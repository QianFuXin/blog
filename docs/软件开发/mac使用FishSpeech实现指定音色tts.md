---
tags: ["mac", "FishSpeech", "tts"]
---

**下述推理代码都使用cpu进行处理（使用mps走不通** ）

> 参考
>
> https://speech.fish.audio/zh/inference/

# 环境与模型

    git clone https://github.com/fishaudio/fish-speech.git
    cd fish-speech
    # 环境
    conda create -n fish-speech python=3.10
    conda activate fish-speech
    # 依赖包
    pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1
    pip install -e ".[stable]"
    # 下载模型
    HF_ENDPOINT=https://hf-mirror.com huggingface-cli download fishaudio/fish-speech-1.5 --local-dir checkpoints/fish-speech-1.5

# 基本流程

## 提取音色

下面的paimon.wav也可以是mp3文件，最终会生成一个`fake.npy`

    python tools/vqgan/inference.py --device cpu \
        -i "paimon.wav" \
        --checkpoint-path "checkpoints/fish-speech-1.5/firefly-gan-vq-fsq-8x1024-21hz-generator.pth"

## 生成语音

    python tools/llama/generate.py --device cpu \
        --text "你要生成的文本" \
        --prompt-text "你提供语音中的文本" \
        --prompt-tokens "fake.npy" \
        --checkpoint-path "checkpoints/fish-speech-1.5" \
        --num-samples 1 \

会生成一个codes_0.npy，下面将codes_0.npy转换为音频文件

    python tools/vqgan/inference.py --device cpu \
        -i "codes_0.npy" \
        --checkpoint-path "checkpoints/fish-speech-1.5/firefly-gan-vq-fsq-8x1024-21hz-generator.pth"

**最后生成目标文件：generated_audio.wav**

# api接口

## 服务端

需要手动修改代码，因为mac端使用mps推理莫名出错

![](/images/mac使用FishSpeech实现指定音色tts.png)

    python -m tools.api_server \
        --listen 0.0.0.0:8080 \
        --llama-checkpoint-path "checkpoints/fish-speech-1.5" \
        --decoder-checkpoint-path "checkpoints/fish-speech-1.5/firefly-gan-vq-fsq-8x1024-21hz-generator.pth" \
        --decoder-config-name firefly_gan_vq

## 客户端

**流式传输**

    python -m tools.api_client \
        --text "要输入的文本" \
        --reference_audio "参考音频路径" \
        --reference_text "参考音频的文本内容" \
        --streaming True

**本地保存（生成generated.mp3）**

    python -m tools.api_client \
        --text "要输入的文本" \
        --reference_audio "参考音频路径1" "参考音频路径2" \
        --reference_text "参考音频的文本内容1" "参考音频的文本内容2"\
        --output "generated" \
        --format "mp3"

## 自定义发送请求（服务分离，环境外调用）

    import ormsgpack
    import requests
    from pathlib import Path
    from pydantic import BaseModel, Field, conint, model_validator
    import base64
    from typing_extensions import Annotated
    from typing import Literal

    # 默认输入参数
    default_args = {
        "text": "输入的文本",
        "reference_audio": ["参考音频路径1"],
        "reference_text": ["参考音频的文本内容1"],
        "output": "文件名称",
        "format": "mp3",
        "url": "http://127.0.0.1:8080/v1/tts",
        "api_key": "YOUR_API_KEY",
    }


    class ServeReferenceAudio(BaseModel):
        audio: bytes
        text: str

        @model_validator(mode="before")
        def decode_audio(cls, values):
            audio = values.get("audio")
            if (
                    isinstance(audio, str) and len(audio) > 255
            ):  # Check if audio is a string (Base64)
                try:
                    values["audio"] = base64.b64decode(audio)
                except Exception as e:
                    # If the audio is not a valid base64 string, we will just ignore it and let the server handle it
                    pass
            return values

        def __repr__(self) -> str:
            return f"ServeReferenceAudio(text={self.text!r}, audio_size={len(self.audio)})"


    class ServeTTSRequest(BaseModel):
        text: str
        chunk_length: Annotated[int, conint(ge=100, le=300, strict=True)] = 200
        # Audio format
        format: Literal["wav", "pcm", "mp3"] = "wav"
        # References audios for in-context learning
        references: list[ServeReferenceAudio] = []
        # Reference id
        # For example, if you want use https://fish.audio/m/7f92f8afb8ec43bf81429cc1c9199cb1/
        # Just pass 7f92f8afb8ec43bf81429cc1c9199cb1
        reference_id: str | None = None
        seed: int | None = None
        use_memory_cache: Literal["on", "off"] = "off"
        # Normalize text for en & zh, this increase stability for numbers
        normalize: bool = True
        # not usually used below
        streaming: bool = False
        max_new_tokens: int = 1024
        top_p: Annotated[float, Field(ge=0.1, le=1.0, strict=True)] = 0.7
        repetition_penalty: Annotated[float, Field(ge=0.9, le=2.0, strict=True)] = 1.2
        temperature: Annotated[float, Field(ge=0.1, le=1.0, strict=True)] = 0.7

        class Config:
            # Allow arbitrary types for pytorch related types
            arbitrary_types_allowed = True


    def audio_to_bytes(file_path):
        if not file_path or not Path(file_path).exists():
            return None
        with open(file_path, "rb") as wav_file:
            wav = wav_file.read()
        return wav


    def read_ref_text(ref_text):
        path = Path(ref_text)
        if path.exists() and path.is_file():
            with path.open("r", encoding="utf-8") as file:
                return file.read()
        return ref_text


    # 构造请求数据
    ref_audios = default_args["reference_audio"]
    ref_texts = default_args["reference_text"]
    byte_audios = [audio_to_bytes(ref_audio) for ref_audio in ref_audios]
    ref_texts = [read_ref_text(ref_text) for ref_text in ref_texts]

    data = {
        "text": default_args["text"],
        "references": [
            ServeReferenceAudio(
                audio=ref_audio if ref_audio is not None else b"", text=ref_text
            )
            for ref_text, ref_audio in zip(ref_texts, byte_audios)
        ],
        "normalize": True,
        "format": default_args["format"],
        "max_new_tokens": 1024,
        "chunk_length": 200,
        "top_p": 0.7,
        "repetition_penalty": 1.2,
        "temperature": 0.7,
        "streaming": False,
        "use_memory_cache": "off",
        "seed": None,
    }

    pydantic_data = ServeTTSRequest(**data)

    # 发送 POST 请求
    response = requests.post(
        default_args["url"],
        data=ormsgpack.packb(pydantic_data, option=ormsgpack.OPT_SERIALIZE_PYDANTIC),
        headers={
            "authorization": f"Bearer {default_args['api_key']}",
            "content-type": "application/msgpack",
        },
    )

    # 处理响应
    if response.status_code == 200:
        audio_content = response.content
        audio_path = f"{default_args['output']}.{default_args['format']}"
        with open(audio_path, "wb") as audio_file:
            audio_file.write(audio_content)
        print(f"音频已保存到 '{audio_path}'")
    else:
        print(f"请求失败，状态码: {response.status_code}")
        try:
            print(response.json())
        except Exception as e:
            print(f"响应内容解析失败: {e}")

# docker

目前，mac的m系列没有对应支持的镜像，如果是linux，直接使用下述指令。

    # 拉取镜像
    docker pull fishaudio/fish-speech:latest-dev
    # 运行镜像
    docker run -it \
        --name fish-speech \
        --gpus all \
        -p 7860:7860 \
        fishaudio/fish-speech:latest-dev \
        zsh
    # 进入容器
    docker exec -it fish-speech zsh
    # 下载模型（这个后期可以自己映射路径）
    HF_ENDPOINT=https://hf-mirror.com huggingface-cli download fishaudio/fish-speech-1.5 --local-dir checkpoints/fish-speech-1.5
    # 开启服务
    python -m tools.api_server \
        --listen 0.0.0.0:8080 \
        --llama-checkpoint-path "checkpoints/fish-speech-1.5" \
        --decoder-checkpoint-path "checkpoints/fish-speech-1.5/firefly-gan-vq-fsq-8x1024-21hz-generator.pth" \
        --decoder-config-name firefly_gan_vq

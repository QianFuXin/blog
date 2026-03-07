---
tags: ["python", "录音"]
---

# 代码

```python

    import pyaudio
    import wave
    import threading
    import time


    class AudioRecorder:
        def __init__(self):
            self.p = pyaudio.PyAudio()
            self.stream = None
            self.frames = []
            self.device_index = None
            self.is_recording = False
            self.recording_thread = None

        def list_input_devices(self):
            """列出所有可用的录音设备"""
            device_list = []
            for i in range(self.p.get_device_count()):
                device_info = self.p.get_device_info_by_index(i)
                # 筛选出录音设备
                if device_info["maxInputChannels"] > 0:
                    device_list.append((i, device_info["name"]))
                    print(f"Index: {i}, Name: {device_info['name']}")
            return device_list

        def start_recording(self, device_index, filename="output.wav"):
            """使用指定设备开始录音"""
            self.device_index = device_index
            self.frames = []
            self.is_recording = True

            # 启动录音流
            self.stream = self.p.open(format=pyaudio.paInt16,
                                      channels=1,
                                      rate=44100,
                                      input=True,
                                      input_device_index=device_index,
                                      frames_per_buffer=1024)
            print("Recording started. Call `stop_recording` to end manually.")

            # 启动录音线程
            self.recording_thread = threading.Thread(target=self.record, args=(filename,))
            self.recording_thread.start()

        def record(self, filename):
            """录音线程函数，用于捕获音频数据"""
            try:
                while self.is_recording:
                    data = self.stream.read(1024, exception_on_overflow=False)
                    self.frames.append(data)
            except Exception as e:
                print(f"Error during recording: {e}")

        def stop_recording(self, filename="output.wav"):
            """停止录音并保存文件"""
            if self.is_recording:
                print("Stopping recording...")
                # 停止录音循环
                self.is_recording = False

                # 等待录音线程结束
                if self.recording_thread:
                    self.recording_thread.join()

                # 停止并关闭音频流
                if self.stream:
                    self.stream.stop_stream()
                    self.stream.close()
                    self.stream = None
                print("Recording stopped.")

                # 保存录音文件
                with wave.open(filename, 'wb') as wf:
                    wf.setnchannels(1)
                    wf.setsampwidth(self.p.get_sample_size(pyaudio.paInt16))
                    wf.setframerate(44100)
                    wf.writeframes(b''.join(self.frames))
                print(f"Recording saved as {filename}")

        def close(self):
            """关闭 PyAudio 实例"""
            self.p.terminate()


    # 使用示例
    recorder = AudioRecorder()
    """
    1. 列出所有录音设备
    输出示例
    Index: 0, Name: “钱甫新的iPhone”的麦克风
    Index: 1, Name: MacBook Air麦克风
    """
    recorder.list_input_devices()
    """
    2. 选择一个设备进行录音（此处以设备索引0为例，具体要使用哪个录音设备，请参考具体的【列出所有录音设备】输出结果）
    """
    recorder.start_recording(device_index=0, filename="test_recording.wav")

    # 模拟一些操作或等待录音，用户可以在适当的时候手动停止录音
    time.sleep(3)
    """
    3. 手动接入停止录音
    """
    recorder.stop_recording(filename="test_recording.wav")
    # 最后关闭 PyAudio 实例
    recorder.close()
```

# 环境

    python3.10

    PyAudio==0.2.14

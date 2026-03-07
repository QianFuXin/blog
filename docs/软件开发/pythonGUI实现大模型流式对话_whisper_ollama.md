---
tags: ["python", "whisper", "ollama"]
---

# 服务部署

## 部署whisper api服务

    version: '3.8'

    services:
      whisper-asr:
        image: onerahmet/openai-whisper-asr-webservice:latest
        container_name: whisper-asr-service
        ports:
          - "9099:9000"
        volumes:
          - /Users/qianfuxin/dc/whisper:/root/.cache/whisper
        environment:
          - ASR_MODEL=base
          - ASR_ENGINE=openai_whisper
        restart: unless-stopped

## 部署ollama

见官网

# 代码

    import mimetypes
    import os
    import sys
    from datetime import datetime
    import socksio
    import requests
    from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QLabel, QTextEdit
    import pyaudio
    import wave
    import threading
    from langchain_core.messages import HumanMessage, SystemMessage
    from langchain_core.output_parsers import StrOutputParser
    from langchain_openai import ChatOpenAI


    def transcribe_audio(file_path):
        """
        Function to transcribe audio using the ASR service.

        :param file_path: The full path to the audio file to be transcribed.
        :return: The response text from the ASR service.
        """
        # Ensure the file exists
        if not os.path.isfile(file_path):
            raise FileNotFoundError(f"Audio file not found at: {file_path}")

        # Determine MIME type
        mime_type, _ = mimetypes.guess_type(file_path)
        if mime_type is None:
            raise ValueError(f"Unable to determine MIME type for file: {file_path}")

        # Extract file name
        file_name = os.path.basename(file_path)

        url = "http://localhost:9099/asr"
        params = {
            "encode": "true",
            "task": "transcribe",
            "language": "zh",
            "initial_prompt": "这是一段简体中文",
            "word_timestamps": "false",
            "output": "txt"
        }
        headers = {
            "accept": "application/json",
        }

        # Open the audio file
        with open(file_path, "rb") as audio_file:
            files = {
                "audio_file": (file_name, audio_file, mime_type)
            }

            # Send the POST request
            response = requests.post(url, headers=headers, params=params, files=files)
        return response.text


    def get_model(base_url="http://127.0.0.1:11434/v1"):
        """Get the AI model instance."""
        model = ChatOpenAI(
            api_key="ollama",
            model="qwen2",
            base_url=base_url
        )
        return model


    class AudioRecorder:
        def __init__(self):
            self.p = pyaudio.PyAudio()
            self.stream = None
            self.frames = []
            self.device_index = None
            self.is_recording = False
            self.recording_thread = None

        def list_input_devices(self):
            """List available input devices."""
            device_list = []
            for i in range(self.p.get_device_count()):
                device_info = self.p.get_device_info_by_index(i)
                if device_info["maxInputChannels"] > 0:
                    device_list.append((i, device_info["name"]))
            return device_list

        def start_recording(self, device_index):
            """Start recording with the specified device."""
            self.device_index = device_index
            self.frames = []
            self.is_recording = True

            self.stream = self.p.open(format=pyaudio.paInt16,
                                      channels=1,
                                      rate=44100,
                                      input=True,
                                      input_device_index=device_index,
                                      frames_per_buffer=1024)

            self.recording_thread = threading.Thread(target=self.record)
            self.recording_thread.start()

        def record(self):
            """Record audio in a separate thread."""
            try:
                while self.is_recording:
                    data = self.stream.read(1024, exception_on_overflow=False)
                    self.frames.append(data)
            except IOError as e:
                print(f"Buffer overflow occurred: {e}")

        def stop_recording(self, filename):
            """Stop recording and save the file."""
            if self.is_recording:
                self.is_recording = False
                if self.recording_thread and self.recording_thread.is_alive():
                    self.recording_thread.join()

                if self.stream:
                    self.stream.stop_stream()
                    self.stream.close()
                    self.stream = None

                with wave.open(filename, 'wb') as wf:
                    wf.setnchannels(1)
                    wf.setsampwidth(self.p.get_sample_size(pyaudio.paInt16))
                    wf.setframerate(44100)
                    wf.writeframes(b''.join(self.frames))

        def close(self):
            """Close the PyAudio instance."""
            self.p.terminate()


    class RecorderApp(QWidget):
        def __init__(self):
            super().__init__()

            self.recorder = AudioRecorder()
            self.filename = None
            self.model = get_model()
            self.init_ui()

        def init_ui(self):
            self.setWindowTitle("实时语音识别与问答")
            self.setGeometry(100, 100, 500, 400)

            layout = QVBoxLayout()

            # 开启录音按钮
            self.start_button = QPushButton("开启录音")
            self.start_button.clicked.connect(self.start_recording)
            layout.addWidget(self.start_button)

            # 结束录音按钮
            self.stop_button = QPushButton("结束录音")
            self.stop_button.clicked.connect(self.stop_recording)
            self.stop_button.setEnabled(False)
            layout.addWidget(self.stop_button)

            # 状态显示标签
            self.status_label = QLabel("用户问题: ", self)
            layout.addWidget(self.status_label)

            # 输出结果框
            self.output_box = QTextEdit(self)
            self.output_box.setReadOnly(True)
            layout.addWidget(self.output_box)

            self.setLayout(layout)

        def start_recording(self):
            device_index = 0  # For simplicity, we assume default device.
            start_time = datetime.now().strftime("%Y%m%d_%H%M%S")
            self.filename = f"{start_time}.wav"

            self.recorder.start_recording(device_index)
            self.start_button.setEnabled(False)
            self.stop_button.setEnabled(True)

        def stop_recording(self):
            self.recorder.stop_recording(self.filename)
            self.start_button.setEnabled(True)
            self.stop_button.setEnabled(False)

            # 转录音频并生成回答
            transcription = transcribe_audio(self.filename)
            self.status_label.setText(f"用户问题: {transcription}")
            self.generate_answer(transcription)

        def generate_answer(self, transcription):
            """
            Send the transcription to the large model and display the streamed response.
            """
            QApplication.processEvents()

            messages = [
                SystemMessage("你是一个智能助手。"),
                HumanMessage(content=transcription)
            ]
            parser = StrOutputParser()
            chain = self.model | parser

            response = ""
            for token in chain.stream(messages):
                response += token
                self.output_box.setText(response)
                QApplication.processEvents()


    if __name__ == "__main__":
        app = QApplication(sys.argv)
        window = RecorderApp()
        window.show()
        sys.exit(app.exec_())

### 仅仅实现ASR

    import mimetypes
    import os
    import sys
    from datetime import datetime

    import requests
    from PyQt5.QtGui import QIcon
    from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QLabel, QLineEdit, QComboBox, QMessageBox, \
        QSystemTrayIcon, QMenu, QAction
    import pyaudio
    import wave
    import threading


    def transcribe_audio(file_path):
        """
        Function to transcribe audio using the ASR service.

        :param file_path: The full path to the audio file to be transcribed.
        :return: The response text from the ASR service.
        """
        # Ensure the file exists
        if not os.path.isfile(file_path):
            raise FileNotFoundError(f"Audio file not found at: {file_path}")

        # Determine MIME type
        mime_type, _ = mimetypes.guess_type(file_path)
        if mime_type is None:
            raise ValueError(f"Unable to determine MIME type for file: {file_path}")

        # Extract file name
        file_name = os.path.basename(file_path)

        url = "http://localhost:9099/asr"
        params = {
            "encode": "true",
            "task": "transcribe",
            "language": "zh",
            "initial_prompt": "这是一段简体中文",
            "word_timestamps": "false",
            "output": "txt"
        }
        headers = {
            "accept": "application/json",
        }

        # Open the audio file
        with open(file_path, "rb") as audio_file:
            files = {
                "audio_file": (file_name, audio_file, mime_type)
            }

            # Send the POST request
            response = requests.post(url, headers=headers, params=params, files=files)

        return response.text


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
            return device_list

        def start_recording(self, device_index):
            """使用指定设备开始录音"""
            self.device_index = device_index
            self.frames = []
            self.is_recording = True

            self.stream = self.p.open(format=pyaudio.paInt16,
                                      channels=1,
                                      rate=44100,
                                      input=True,
                                      input_device_index=device_index,
                                      frames_per_buffer=1024)

            # 启动录音线程
            self.recording_thread = threading.Thread(target=self.record)
            self.recording_thread.start()

        def record(self):
            """录音线程函数，用于捕获音频数据"""
            try:
                while self.is_recording:
                    data = self.stream.read(1024, exception_on_overflow=False)
                    self.frames.append(data)
            except IOError as e:
                print(f"Buffer overflow occurred: {e}")

        def stop_recording(self, filename):
            """停止录音并保存文件"""
            if self.is_recording:
                self.is_recording = False
                if self.recording_thread and self.recording_thread.is_alive():
                    self.recording_thread.join()

                if self.stream:
                    self.stream.stop_stream()
                    self.stream.close()
                    self.stream = None

                with wave.open(filename, 'wb') as wf:
                    wf.setnchannels(1)
                    wf.setsampwidth(self.p.get_sample_size(pyaudio.paInt16))
                    wf.setframerate(44100)
                    wf.writeframes(b''.join(self.frames))

        def close(self):
            """关闭 PyAudio 实例"""
            self.p.terminate()


    def resource_path(relative_path):
        """获取资源文件的绝对路径"""
        if hasattr(sys, '_MEIPASS'):
            return os.path.join(sys._MEIPASS, relative_path)
        return os.path.join(os.path.abspath("."), relative_path)


    class RecorderApp(QWidget):
        def __init__(self):
            super().__init__()

            self.recorder = AudioRecorder()
            self.filename = None

            self.init_ui()

        def init_ui(self):
            self.setWindowTitle("录音器")
            self.setGeometry(100, 100, 400, 300)

            # 创建系统托盘图标
            self.tray_icon = QSystemTrayIcon(self)
            self.tray_icon.setIcon(QIcon(resource_path("icon.png")))

            # 创建托盘菜单
            tray_menu = QMenu()

            # 添加显示窗口的选项
            show_action = QAction("显示窗口", self)
            show_action.triggered.connect(self.show)
            tray_menu.addAction(show_action)

            # 添加退出的选项
            exit_action = QAction("退出", self)
            exit_action.triggered.connect(QApplication.instance().quit)
            tray_menu.addAction(exit_action)

            # 将菜单添加到托盘图标
            self.tray_icon.setContextMenu(tray_menu)

            # 设置点击双击托盘图标的动作
            self.tray_icon.activated.connect(self.on_tray_icon_activated)

            # 显示托盘图标
            self.tray_icon.show()

            layout = QVBoxLayout()

            # 开启录音按钮
            self.start_button = QPushButton("开启录音")
            self.start_button.clicked.connect(self.start_recording)
            layout.addWidget(self.start_button)

            # 结束录音按钮
            self.stop_button = QPushButton("结束录音")
            self.stop_button.clicked.connect(self.stop_recording)
            self.stop_button.setEnabled(False)
            layout.addWidget(self.stop_button)

            # 设备选择下拉菜单
            self.device_selector = QComboBox(self)
            devices = self.recorder.list_input_devices()
            if devices:
                for index, name in devices:
                    self.device_selector.addItem(name, index)
            else:
                QMessageBox.warning(self, "错误", "未检测到录音设备")
                self.start_button.setEnabled(False)
            layout.addWidget(self.device_selector)

            # 状态显示标签
            self.status_label = QLabel("状态: 未录音", self)
            layout.addWidget(self.status_label)

            self.setLayout(layout)

        def closeEvent(self, event):
            """窗口关闭事件"""
            if self.isVisible():  # 如果窗口可见，则最小化到托盘
                event.ignore()
                self.hide()
                self.tray_icon.showMessage(
                    "应用最小化", "程序已最小化到托盘",
                    QSystemTrayIcon.Information, 2000
                )
            else:  # 关闭应用时清理资源
                self.recorder.close()
                event.accept()

        def on_tray_icon_activated(self, reason):
            """托盘图标激活事件"""
            if reason == QSystemTrayIcon.DoubleClick:
                self.show()

        def start_recording(self):
            device_index = self.device_selector.currentData()
            start_time = datetime.now().strftime("%Y%m%d_%H%M%S")
            self.filename = f"{start_time}.wav"

            self.recorder.start_recording(device_index)
            self.status_label.setText("状态: 录音中...")
            self.start_button.setEnabled(False)
            self.stop_button.setEnabled(True)

        def stop_recording(self):
            end_time = datetime.now().strftime("%Y%m%d_%H%M%S")
            self.filename = f"{self.filename[:-4]}-{end_time}.wav"  # 更新文件名
            self.recorder.stop_recording(self.filename)
            self.status_label.setText(f"状态: 录音已保存为 {self.filename}")
            self.start_button.setEnabled(True)
            self.stop_button.setEnabled(False)
            self.status_label.setText(f"状态: 录音已保存为 {self.filename}\n录音内容：{transcribe_audio(self.filename)}")


    # 主程序入口
    if __name__ == "__main__":
        app = QApplication(sys.argv)
        window = RecorderApp()
        window.show()
        sys.exit(app.exec_())
        """
        pyinstaller --onefile --windowed --noconfirm --add-data "icon.png;." app.py
        """

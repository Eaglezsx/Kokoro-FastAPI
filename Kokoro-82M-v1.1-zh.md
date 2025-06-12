switch the model to hexgrad/Kokoro-82M-v1.1-zh
https://github.com/remsky/Kokoro-FastAPI/issues/214
------
将github原工程按照这个步骤来，首先这个可能会导致你原有的环境不可用

1.获取资源
sudo apt install espeak-ng

git lfs install
cd api/src/models
git clone https://huggingface.co/hexgrad/Kokoro-82M-v1.1-zh
mv Kokoro-82M-v1.1-zh v1_1-zh

cp -r v1_1-zh/voices ../voices/v1_1-zh

pip uninstall kokoro
pip install kokoro
pip install misaki[zh]
2.修改代码
api/src/core/config.py
    allow_local_voice_saving: bool = (
        False  # Whether to allow saving combined voices locally
    )
改为

    allow_local_voice_saving: bool = (
        False  # Whether to allow saving combined voices locally
    )
    repo_id: str = "hexgrad/Kokoro-82M"
api/src/core/model_config.py
    # Model filename
    pytorch_kokoro_v1_file: str = Field(
        "v1_0/kokoro-v1_0.pth", description="PyTorch Kokoro V1 model filename"
    )
改为

    # Model filename
    pytorch_kokoro_v1_file: str = Field(
        "v1_1-zh/kokoro-v1_1-zh.pth", description="PyTorch Kokoro V1 model filename"
    )
api/src/inference/kokoro_v1.py
            # 第一块
            self._model = KModel(config=config_path, model=model_path).eval()

            # 第二块
            self._pipelines[lang_code] = KPipeline(
                lang_code=lang_code, model=self._model, device=self._device
            )
改为

            # 第一块
            self._model = KModel(config=config_path, model=model_path, repo_id=settings.repo_id).eval()


            # 第二块
            self._pipelines[lang_code] = KPipeline(
                lang_code=lang_code, model=self._model, device=self._device, repo_id=settings.repo_id
            )
api/src/inference/model_manager.py
                warmup_text = "Warmup text for initialization."
改为

                warmup_text = "初始化的预热文本。"
api/src/services/text_processing/phonemizer.py
    if language not in lang_map:
        raise ValueError(f"Unsupported language code: {language}")

    return EspeakBackend(lang_map[language])
    
改为

    return EspeakBackend("cmn")
3.修改启动脚本
export VOICES_DIR=src/voices/v1_1-zh
export DEFAULT_VOICE=zf_094
export REPO_ID=hexgrad/Kokoro-82M-v1.1-zh
-----
修复中英文混合不生成语音的问题
https://github.com/remsky/Kokoro-FastAPI/pull/237
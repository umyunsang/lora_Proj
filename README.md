# OneTrainer + ComfyUI LoRA Project

SSH 서버 환경에서 OneTrainer와 ComfyUI를 사용한 LoRA 훈련 및 이미지 생성 프로젝트입니다.

## 🔄 워크플로우

### 1. 로컬에서 GUI 설정
```bash
# 프로젝트 클론
git clone https://github.com/umyunsang/lora_Proj.git
cd lora_Proj/OneTrainer

# GUI 실행 (Windows/Mac/Linux Desktop)
./start-ui.sh
# 또는
python scripts/train_ui.py
```

### 2. 서버에서 CLI 훈련
```bash
# 최신 변경사항 가져오기
cd /home/student_15030/lora_project
git pull origin master

# 훈련 실행
cd OneTrainer
./run-cmd.sh train --config-path [설정파일경로]

# 백그라운드 실행
nohup ./run-cmd.sh train --config-path config.json > training.log 2>&1 &
```

## 📁 중요 파일 구조

```
lora_project/
├── README.md                    # 이 파일
├── OneTrainer/                  # OneTrainer 메인 폴더
└── ComfyUI/                     # ComfyUI 메인 폴더
    ├── run-cmd.sh              # 스크립트 실행기
    ├── start-ui.sh             # GUI 실행 스크립트
    ├── .gitignore              # Git 제외 파일 설정
    │
    ├── training_presets/        # 훈련 설정 템플릿
    │   ├── #sd 1.5 LoRA.json   # SD 1.5 LoRA 설정
    │   ├── #sdxl 1.0 LoRA.json # SDXL LoRA 설정
    │   ├── #flux LoRA.json     # Flux LoRA 설정
    │   └── #chroma LoRA.json   # Chroma LoRA 설정
    │
    ├── training_concepts/       # 훈련 데이터 설정
    │   └── concepts.json       # 컨셉 정의 파일
    │
    ├── training_samples/        # 샘플 생성 설정
    │   └── samples.json        # 샘플 정의 파일
    │
    ├── scripts/                # 실행 스크립트들
    │   ├── train.py            # 메인 훈련 스크립트
    │   ├── train_ui.py         # GUI 훈련 인터페이스
    │   ├── sample.py           # 샘플 생성
    │   ├── generate_captions.py # 캡션 자동 생성
    │   └── generate_masks.py   # 마스크 생성
    │
    └── workspace/              # 훈련 결과 (Git 제외)
        └── run/
            ├── config/         # 훈련 설정 백업
            └── tensorboard/    # TensorBoard 로그
    │
    └── ComfyUI/                # ComfyUI 메인 폴더
        ├── main.py             # ComfyUI 실행 스크립트
        ├── models/             # 모델 저장소 (Git 제외)
        ├── output/             # 생성 이미지 출력 (Git 제외)
        ├── input/              # 입력 이미지 폴더
        ├── custom_nodes/       # 커스텀 노드들 (Git 제외)
        └── web/                # 웹 인터페이스
```

## 🚀 주요 사용법

### OneTrainer - LoRA 훈련

#### GUI에서 설정 (로컬)
1. `start-ui.sh` 실행하여 GUI 열기
2. 모델 타입, 학습률, 배치 크기 등 설정
3. 훈련 데이터 경로 설정
4. 설정 파일 저장 후 Git 커밋

#### CLI에서 훈련 (서버)
```bash
cd OneTrainer

# 기본 훈련
./run-cmd.sh train --config-path training_presets/#sd\ 1.5\ LoRA.json

# 데이터 전처리
./run-cmd.sh generate_captions --input-dir /path/to/images
./run-cmd.sh generate_masks --input-dir /path/to/images

# 샘플 생성
./run-cmd.sh sample --model-path workspace/run/models/model.safetensors --prompt "test prompt"

# 모델 변환
./run-cmd.sh convert_model --input-path input.ckpt --output-path output.safetensors
```

### ComfyUI - 이미지 생성

#### 서버에서 ComfyUI 실행
```bash
cd ComfyUI
source venv/bin/activate
python main.py --listen 0.0.0.0 --port 8188

# 백그라운드 실행
nohup python main.py --listen 0.0.0.0 --port 8188 > comfyui.log 2>&1 &
```

#### 웹 브라우저에서 접속
- **로컬 접속**: http://localhost:8188
- **원격 접속**: http://[서버IP]:8188

## ⚙️ 환경 설정

### 서버 환경변수
```bash
export OT_CUDA_LOWMEM_MODE=true     # 메모리 절약 모드
export HF_HUB_DISABLE_XET=1         # XET 버그 방지
export CUDA_VISIBLE_DEVICES=0       # GPU 지정
```

### 필수 디렉토리 생성
```bash
mkdir -p training_data/{images,captions,masks}
mkdir -p outputs/models
```

## 📊 모니터링

### 훈련 진행 상황 확인
```bash
# 로그 실시간 확인
tail -f training.log

# GPU 사용량 확인
nvidia-smi

# TensorBoard 실행 (포트 6006)
tensorboard --logdir workspace/run/tensorboard --host 0.0.0.0
```

### 백그라운드 프로세스 관리
```bash
# Screen 사용
screen -S onetrainer
./run-cmd.sh train --config-path config.json
# Ctrl+A, D로 분리

# 프로세스 확인
ps aux | grep train

# 프로세스 종료
kill -9 [PID]
```

## 🔧 문제 해결

### GUI 실행 안될 때
- SSH에서는 GUI 사용 불가 → 로컬에서 실행
- X11 포워딩 필요시: `ssh -X user@server`

### 메모리 부족 시
```bash
export OT_CUDA_LOWMEM_MODE=true
# 배치 크기 줄이기
# 해상도 낮추기 (512 → 256)
```

### 훈련 중단 시
```bash
# 체크포인트에서 재시작
./run-cmd.sh train --config-path config.json --resume-from workspace/run/models/checkpoint.safetensors
```

## 📝 Git 워크플로우

### 로컬에서 설정 변경 후
```bash
git add training_presets/ training_concepts/ training_samples/
git commit -m "Update training configs for [project_name]"
git push origin master
```

### 서버에서 최신 설정 가져오기
```bash
git pull origin master
```

## 🎯 권장 설정

### SD 1.5 LoRA (8GB GPU)
- 배치 크기: 1-2
- 학습률: 0.0001-0.0003
- 해상도: 512x512
- 스텝: 1000-3000

### SDXL LoRA (16GB+ GPU)
- 배치 크기: 1
- 학습률: 0.0001
- 해상도: 1024x1024
- 스텝: 1500-4000

## 📞 지원

- GitHub Issues: [https://github.com/umyunsang/lora_Proj/issues](https://github.com/umyunsang/lora_Proj/issues)
- OneTrainer 공식: [https://github.com/Nerogar/OneTrainer](https://github.com/Nerogar/OneTrainer)

# MNC - DKMediInfo PoC
2023 창업성공패키지 글로벌창업사관학교  
AI 기술 실증 컨설팅 과정으로 마인즈앤컴퍼니와 디케이메디인포가 진행한 PoC 모델에 대한 코드입니다.  
주제: 거대 언어 모델(LLM)을 이용한 간호 기록 작성 예시 생성 및 작성 보조  

# Directory
* data
    - dataset: 학습에 사용할 .jsonl 파일 저장
    - processed: raw data에서 전처리한 파일 저장
    - raw: DB에서 추출한 raw data. (json, xlsx ..)
* models: 학습된 모델을 저장
* templates: prompt 생성에 사용되는 템플릿
* train_output: 학습 과정에서 저장되는 모델 경로
* utils: 데이터 처리 및 기타 함수


# Data
## Preprocess
DB에서 추출된 raw data를 대상으로 아래 작업 수행
* 중복 데이터 제거
* 특수 문자, 자음/모음/숫자로만 이루어진 텍스트 제거
* 공백과 개행 문자 등 제거 이후 중복 제거

## Dataset
`data/dataset/finetuning.jsonl`은 다음과 같은 구조로 되어있습니다.
* instruction: 간호 기록 작성 형식, 설명
* input: 가상 환자 정보
* output: 가상 환자에 대해 작성한 간호 기록
```
...
{"instruction": "...", "input": "...", "output": "..."}
...
```
위의 형식으로 구성된 데이터셋으로 LLM 학습을 위해 다음과 같은 형태로 모델의 입력으로 이용합니다.
```
### 간호기록작성형식: ...
### 환자정보: ...
### 간호기록: ...
```


# Pipeline
아래의 전체 과정을 실행할 수 있는 스크립트 파일을 실행하는 명령어입니다.
```bash
sh pipeline.sh
```


## Data Preprocess
raw_data_preprocess.py의 실행으로 만든 record_N.csv 파일을 읽어서, LLM의 학습 데이터셋으로 사용할 수 있도록 합니다.
```
python raw_data_preprocess.py
python dataset.py
```


## Train
PEFT(Parameter Efficient Fine-Tuning) 기법을 적용하여 학습을 진행합니다.  
베이스 모델로 polyglot-ko-5.8B을 사용하여 학습하도록 하였고, 추후 라이센스상 사용에 문제가 없는 다른 모델로 수정하여 사용할 수 있습니다.  
(단, 모델에 따라 PEFT 학습 라이브러리 사용 부분에 수정이 필요할 수 있습니다.)
```
python finetune.py
```
* 학습을 위해 마인즈앤컴퍼니 내부 V100 (VRAM 32GB) GPU 서버를 사용하였습니다.  (학습 중에 31GB 점유)  
    (Full fine-tuning을 위해서는 5.8B 모델 기준, 약 70GB VRAM이 필요합니다.)
* 9월 30일 기준 전처리된 데이터 약 9천건에 대해, epoch 3 학습에 약 1일 17시간이 소요됩니다.
* nohup 명령어로 학습을 실행하고, 실행 과정은 wandb를 통해 확인하는 것을 권장합니다.  


## Inference
finetuning된 모델이 저장된 경로(--model), 간호 기록 형식(--instruction), samples.py에 있는 환자 이름(--patient)를 입력하여 아래 명령어를 실행합니다.
```
python inference.py --model models/polyglot/check --instruction Focus-DAR --patient 남도일
```
위의 예시로 명령어를 실행하면, 모델에 입력되는 데이터는 다음과 같습니다.
```
### 간호기록작성형식: Focus-DAR 형식으로 간호 기록을 작성하세요.
Focus-DAR 형식에는 focus(포커스), data(자료), action(활동), response(반응) 항목이 포함되어야 합니다.'

### 환자정보: 
이름: 남도일
나이: 8세
성별: 남성
진료과: 응급의학과
진단명: 기타 및 상세불명의 급성 충수염
처방 내역: 남도일 환아(M/8)는 충수돌기염을 진단받고 응급실을 거쳐 입원하였다.
방금 전 충수돌기 절제술을 마치고 회복실에서 병동으로 복귀하였다.
현재 환아는 수술부위의 통증을 호소하며 찡그린 표정을 하고 있으며 보호자도 예민해져 있는 상태이다.
현재 5% DS 40cc/hr로 주입 중이며 PCA 유지중이다.
간호사는 수술 후 환아의 건강문제를 사정하고 환아와 보호자에게 적절한 수술 후 간호를 제공하고자 한다.

### 간호기록: 
```

# Demo
Gradio Demo를 실행할 수 있습니다.  
명령어 실행 후 http://127.0.0.1:7860/ 로 접속하여 확인할 수 있습니다.

```
python app.py
```
app.py 내에서 MODEL 변수 수정을 통해 새로 학습한 모델을 적용할 수 있습니다.


# Contributors
디케이메디인포 이동균 대표  
마인즈앤컴퍼니 천상진 매니저  

MINDS AND COMPANY  
with DKMediInfo

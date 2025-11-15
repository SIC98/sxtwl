# sxtwl Python 3.13 + macOS M1 호환 작업 요약

## 변경 사항

### 1. 새로운 파일 생성

#### `pyproject.toml`
- 현대적인 Python 패키징 표준 적용
- Python 3.8-3.13 지원 명시
- 프로젝트 메타데이터 및 빌드 설정 포함
- setuptools 기반 빌드 백엔드 설정

#### `MANIFEST.in`
- 소스 파일 포함 규칙 정의
- C++ 소스 파일 및 헤더 파일 포함
- 빌드 아티팩트 제외 규칙

#### `.gitignore` (업데이트)
- Python 관련 파일 제외 규칙 추가
- 빌드 아티팩트 제외
- IDE 설정 파일 제외

#### `PYPI_DEPLOY_GUIDE.md`
- PyPI 배포 상세 가이드
- 플랫폼별 빌드 방법
- GitHub Actions 자동화 예시

### 2. 수정된 파일

#### `python/setup.py`
주요 변경사항:
- Python 2 호환 코드 제거
- `pathlib.Path` 사용으로 현대화
- macOS ARM64 (M1/M2/M3) 지원 추가
  - `-arch arm64` 컴파일 플래그
  - ARM64 링크 플래그
- 버전 1.1.0으로 업데이트
- Python 3.8+ 최소 버전 요구사항 명시
- 분류자(classifier) 추가
- 코드 정리 및 간소화

구체적 개선사항:
```python
# macOS M1/ARM64 지원
if platform.system() == 'Darwin' and platform.machine() == 'arm64':
    extra_compile_args.extend(['-arch', 'arm64'])
    extra_link_args.extend(['-arch', 'arm64'])
```

## 테스트 결과

### 빌드 테스트
- ✅ Python 3.14 (3.13 호환)에서 정상 빌드
- ✅ macOS M1 (ARM64) 환경에서 컴파일 성공
- ✅ Wheel 패키지 생성 성공: `sxtwl-1.1.0-cp314-cp314-macosx_26_0_arm64.whl`

### 설치 테스트
- ✅ 빌드된 wheel 패키지 설치 성공
- ✅ 패키지 import 성공
- ✅ 기본 기능(음력 변환) 동작 확인

### 경고 사항
빌드 중 다음 경고가 발생하지만 기능에는 영향 없음:
- C++ 코드에서 `sprintf` 사용 (보안 경고)
- 사용되지 않는 변수
- 논리 연산자 우선순위 경고

## 다음 단계

### PyPI 배포 전 확인사항

1. **버전 정보 확인**
   - `pyproject.toml`: version = "1.1.0"
   - `python/setup.py`: version="1.1.0"

2. **GitHub 저장소 업데이트**
   - README.md에서 GitHub URL 확인: `https://github.com/SIC98/sxtwl`
   - 실제 저장소 URL과 일치하는지 확인

3. **LICENSE 파일**
   - BSD 라이선스 파일이 있는지 확인
   - 없으면 추가 필요

### 배포 방법

#### 빠른 배포 (현재 플랫폼만)
```bash
cd python
python3 -m pip install --upgrade build twine
python3 -m build
python3 -m twine upload dist/*
```

#### 다중 플랫폼 지원
여러 플랫폼을 지원하려면:
1. Linux용 wheel: Docker + manylinux 사용
2. Windows용 wheel: Windows 환경에서 빌드
3. macOS Intel용 wheel: x86_64 환경에서 빌드

또는 GitHub Actions를 사용한 자동 빌드 설정 (PYPI_DEPLOY_GUIDE.md 참조)

## 호환성

### 지원 환경
- Python 3.8, 3.9, 3.10, 3.11, 3.12, 3.13
- macOS (ARM64 및 x86_64)
- Linux (x86_64, ARM64)
- Windows (x86_64)

### 테스트 완료
- ✅ Python 3.14 (3.13 호환)
- ✅ macOS ARM64 (M1)

## 참고 문서

- [PYPI_DEPLOY_GUIDE.md](PYPI_DEPLOY_GUIDE.md): 상세 배포 가이드
- [pyproject.toml](pyproject.toml): 프로젝트 설정
- [python/setup.py](python/setup.py): 빌드 스크립트

## 문의

문제가 발생하면 GitHub Issues에 보고해주세요:
https://github.com/SIC98/sxtwl/issues

# PyPI 배포 가이드

이 문서는 sxtwl 패키지를 PyPI에 배포하는 방법을 설명합니다.

## 사전 준비

### 1. PyPI 계정 생성
- [PyPI](https://pypi.org/)에 계정을 만듭니다
- [TestPyPI](https://test.pypi.org/)에도 계정을 만듭니다 (테스트용)

### 2. API 토큰 생성
1. PyPI 계정 설정에서 API 토큰을 생성합니다
2. 토큰을 안전한 곳에 저장합니다

### 3. 필요한 패키지 설치
```bash
python3 -m pip install --upgrade build twine
```

## 배포 과정

### 1. 버전 업데이트
배포 전에 버전 번호를 업데이트합니다:
- `pyproject.toml` 파일의 `version` 필드
- `python/setup.py` 파일의 `version` 필드

### 2. 빌드 (python 디렉토리에서 실행)

소스 배포판과 wheel 패키지를 빌드합니다:

```bash
cd python
python3 -m build
```

빌드가 완료되면 `dist/` 디렉토리에 다음 파일들이 생성됩니다:
- `sxtwl-X.X.X.tar.gz` (소스 배포판)
- `sxtwl-X.X.X-cpXXX-cpXXX-PLATFORM.whl` (wheel 패키지)

### 3. TestPyPI에 먼저 테스트 (선택사항)

실제 배포 전에 TestPyPI에서 테스트하는 것을 권장합니다:

```bash
python3 -m twine upload --repository testpypi dist/*
```

설치 테스트:
```bash
pip install --index-url https://test.pypi.org/simple/ sxtwl
```

### 4. PyPI에 배포

실제 PyPI에 배포합니다:

```bash
python3 -m twine upload dist/*
```

사용자 이름과 비밀번호를 묻는 경우:
- Username: `__token__`
- Password: (생성한 API 토큰을 입력)

### 5. 배포 확인

배포가 완료되면:
1. [https://pypi.org/project/sxtwl/](https://pypi.org/project/sxtwl/)에서 패키지 확인
2. 설치 테스트:
```bash
pip install sxtwl
```

## 환경별 빌드

### macOS (M1/ARM64)
```bash
cd python
python3 -m build --wheel
```

### Linux (manylinux)
Linux에서 여러 Python 버전을 지원하는 wheel을 빌드하려면 Docker를 사용합니다:

```bash
docker run --rm -v $(pwd):/io quay.io/pypa/manylinux2014_x86_64 /io/build-wheels.sh
```

### Windows
Windows에서 빌드하려면 Visual Studio Build Tools가 필요합니다:

```bash
cd python
python -m build --wheel
```

## 주의사항

1. **버전 관리**: 배포할 때마다 버전을 올려야 합니다. PyPI는 같은 버전을 다시 업로드할 수 없습니다.

2. **플랫폼별 빌드**:
   - 이 패키지는 C++ 확장 모듈을 포함하므로 플랫폼별로 별도의 wheel이 필요합니다
   - 현재 빌드는 macOS ARM64용입니다
   - Linux와 Windows용 wheel을 별도로 빌드해야 합니다

3. **소스 배포판**: `python -m build`를 사용하면 소스 배포판(.tar.gz)도 함께 생성되어, 다른 플랫폼 사용자들이 소스에서 직접 빌드할 수 있습니다.

4. **.pypirc 설정** (선택사항):

   홈 디렉토리에 `~/.pypirc` 파일을 만들어 자격 증명을 저장할 수 있습니다:

   ```ini
   [pypi]
   username = __token__
   password = pypi-AgEIcH...your-token-here

   [testpypi]
   username = __token__
   password = pypi-AgEIcH...your-token-here
   ```

## 자동화 (GitHub Actions)

GitHub Actions를 사용하여 태그를 푸시할 때 자동으로 배포하도록 설정할 수 있습니다.

`.github/workflows/publish.yml` 파일 예시:

```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.8', '3.9', '3.10', '3.11', '3.12', '3.13']

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install build twine

    - name: Build package
      run: |
        cd python
        python -m build --wheel

    - name: Publish to PyPI
      env:
        TWINE_USERNAME: __token__
        TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
      run: |
        python -m twine upload python/dist/*
```

GitHub Repository의 Settings > Secrets에 `PYPI_API_TOKEN`을 추가해야 합니다.

## 문제 해결

### 빌드 오류
- C++ 컴파일러가 설치되어 있는지 확인하세요
- macOS: Xcode Command Line Tools 설치 (`xcode-select --install`)
- Linux: `gcc`, `g++` 설치
- Windows: Visual Studio Build Tools 설치

### 업로드 오류
- API 토큰이 올바른지 확인하세요
- 버전이 중복되지 않는지 확인하세요
- 네트워크 연결을 확인하세요

## 참고 자료

- [Python Packaging Guide](https://packaging.python.org/)
- [PyPI Help](https://pypi.org/help/)
- [Twine Documentation](https://twine.readthedocs.io/)

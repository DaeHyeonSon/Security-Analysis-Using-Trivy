# Security-Analysis-Using-Trivy
**Trivy**를 활용한 Docker 이미지 **취약점 분석** 

<h2 style="font-size: 25px;"> TEAM 👨‍👨‍👧 <br>
</h2>

|<img src="https://avatars.githubusercontent.com/u/81280628?v=4" width="100" height="100"/>|<img src="https://avatars.githubusercontent.com/u/86951396?v=4" width="100" height="100"/>
|:-:|:-:|
|[@손대현](https://github.com/DaeHyeonSon)|[@이아영](https://github.com/ayleeee)|
---

### 개요 🚩
취약점 스캐너로써 잘 알려진 **Trivy**를 활용하여 **프로그램 배포 시 취약점**을 **진단**하며 이를 **분석**해보고자 한다. 

Trivy의 구조는 다음과 같다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/99410af8-3056-4512-8c1d-b80c7c4ad161" width="50%">
</div>

### 취약점 진단 툴 사용 이유 🙄

<details>
<summary>Trivy 사용 이유?</summary>
<div markdown="1">

가장 대중성이 높고 k8s까지 점검 가능한 높은 확장성을 가진 이유때문에 Clair, Anchore 과 같은 취약점 도구 툴이 아닌 Trivy를 선택하여 취약점 진단을 진행하였다. 

</div>
</details>

기본 이미지 내에 여러 취약점이 존재하는데, 컨테이너는 이러한 이미지들을 통해 파생된다. 이때 배포 전 취약점을 사전에 발견하여 보안 사고를 예방하고, 보안 문제를 조기에 발경함으로써 후에 발생 가능한 비용 높은 보안 문제에 대한 사고를 방지할 수 있다.

### 특징


<details>
<summary>OPEN</summary>
<div markdown="1">

- 다양한 스캔 대상 지원: 컨테이너 이미지, 파일 시스템, 코드 리포지토리, IaC 파일 등 다양한 대상을 스캔할 수 있습니다.
  
- 빠른 스캔 속도: Trivy는 캐싱 메커니즘을 통해 스캔 속도를 향상시킵니다.
  
- 광범위한 취약점 데이터베이스: CVE, NVD 등 여러 데이터베이스와 연동하여 최신 취약점을 탐지합니다.
  
- 간편한 통합: GitLab CI/CD 파이프라인에 쉽게 통합할 수 있는 유연한 설정 옵션 제공.
  
- 다양한 출력 형식: JSON, 테이블 등 다양한 형식으로 스캔 결과를 출력하여 후속 처리 용이.
  
- 플러그인 아키텍처: 필요에 따라 기능을 확장할 수 있는 플러그인 지원.

</div>
</details>


### 스캔 가능 범위 

- Container 이미지
- 파일 시스템
- git repository
- VM 이미지
- Kubernetes
- AWS

## Trivy 실습 🔥

### Docker 이미지 취약점 진단

<br>

**[1] NGINX 취약 버전을 통한 실습**
먼저 Trivy 설치를 진행

```bash
sudo apt-get update
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy
```

취약점이 포함된 Nginx Docker 이미지 생성

1. test 폴더 생성
```bash
mkdir test-nginx
cd test-nginx 
```
2. Dokerfile 생성
```bash
# 취약한 버전의 Alpine Linux 사용 (3.12)
FROM alpine:3.12

# 사용 가능한 버전의 Nginx 설치
RUN apk add --no-cache nginx=1.18.0-r3

# 커스텀 index.html 파일 추가
COPY index.html /usr/share/nginx/html/index.html

# 포트 80 노출
EXPOSE 80

# Nginx 실행
CMD ["nginx", "-g", "daemon off;"]
```

3. index.html 생성 
```html
<!DOCTYPE html>
<html>
<head>
    <title>Vulnerable Nginx Page</title>
</head>
<body>
    <h1>Welcome to Vulnerable Nginx!</h1>
    <p>This Nginx server has intentional vulnerabilities for testing purposes.</p>
</body>
</html>
```
4. Docker 이미지 빌드
```bash
docker build -t my-test-nginx:latest .
```

```bash
trivy image my-test-nginx:latest
```

<details>
<summary>성공 시 결과</summary>
<div markdown="1">

<div align="center">
  <img src="https://github.com/user-attachments/assets/9e99e97e-88ec-4fbf-9a5e-b7a8da382a25" width ="50%">
</div>

</div>
</details>

<details>
<summary>실패 시 결과</summary>
<div markdown="1">

<div align="center">
  <img src="https://github.com/user-attachments/assets/78d03b90-e941-48c5-a063-8404d4720212" width ="50%">
</div>

<br>

※ `Alpine 3.12` 리포지토리에 해당 버전이 존재하지 않아 `run`단계에서 오류가 발생 가능하다 -> 버전을 맞춰주면 해결 가능
</div>
</details>

5. Trivy를 통한 취약점 스캔
취약점이 포함된 nginx 패키지 버전을 통하여 취약점을 스캔한 결과는 다음과 같다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/196f9455-9eb9-4f67-80c0-b14ec1e474fd" width ="50%">
</div>

<br>

**취약점 분석 결과** 

실제 취약점 번호를 통해 조회 해본 결과 `CVE-2022-37434` 취약점은 **zlib 버퍼 오버플로우/오버리드** 취약점으로써 zlib 라이브러리의 inflate 함수 내 inflate.c 파일에서 발생하는 힙 기반 버퍼 오버리드 또는 버퍼 오버플로우 취약점인 것을 확인하였다.

해당 취약점의 원인은 zlib 라이브러리의 `inflate` 함수를 압축 해제할 시 큰 gzip 헤더의 추가 필드를 처리할 때, 충분한 버퍼 크기를 확인하지 않아 버퍼의 경계를 넘어서는 데이터 접근이 발생할 수 있는데 이때 발생 가능한 취약점이다.   

**심각도**는 높음(High) 단계로써 메모리 손상, 래플리케이션 충돌 등의 문제를 야기할 수 있으며, **대응방안으로는** zlib의 최신 버전으로 업데이트, 영향 받는 애플리케이션 식별 및 업데이트, 보안 설정 강화와 같은 방법이 존재한다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/b4794275-e03e-4a0a-8d08-277b7cef4257" width ="50%">
</div>

<br>

* * *

**[2] WOOSO 이미지의 취약점 찾기**
<br>
[1] 의 실습을 바탕으로 [WOOSO](https://github.com/DaeHyeonSon/WhiteClothesPeople.git)의 취약점을 파악해보았다. 

1. WOOSO 이미지 파일 제작
```dockerfile
FROM gradle:7.6.1-jdk17 AS build

WORKDIR /app

COPY build.gradle settings.gradle ./

COPY src ./src

RUN gradle build --no-daemon

FROM openjdk:17-jdk-slim

WORKDIR /app

COPY --from=build /app/build/libs/wooso-0.0.1-SNAPSHOT.jar ./wooso.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "wooso.jar"]

```
2. Trivy로 이미지 취약점 스캔<br>
> 나온 결과값을 vulnerabilites.json 파일로 내보낸다.
```cmd
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/output aquasec/trivy:latest image --format json -o /output/vulnerabilities.json wooso

```
3. 결과값을 모두가 볼 수 있도록 issue 탭에 등록<br>
> 편의상 10개만 등록되도록 설정하였다. 
```python
import json
import requests

with open("vulnerabilities.json", 'r') as file:
    data = json.load(file)

GITHUB_REPO = "username/repo-name"
GITHUB_TOKEN = "token"

GITHUB_API_URL = f"https://api.github.com/repos/{GITHUB_REPO}/issues"

# Function to create an issue on GitHub
def create_github_issue(title, body):
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github.v3+json"
    }

    issue_data = {
        "title": title,
        "body": body,
        "labels": ["vulnerability"]
    }

    response = requests.post(GITHUB_API_URL, json=issue_data, headers=headers)

    if response.status_code == 201:
        print(f"Issue created: {title}")
    else:
        print(f"Failed to create issue: {response.content}")

max_issues = 10
issue_count = 0

for result in data['Results']:
    for vulnerability in result.get('Vulnerabilities', []):
        if issue_count >= max_issues:
            break
        title = f"Vulnerability: {vulnerability.get('VulnerabilityID', 'No ID')} in {vulnerability.get('PkgName', 'Unknown Package')}"
        description = vulnerability.get('Description', 'No description available.')
        body = f"""
        **Package:** {vulnerability.get('PkgName', 'Unknown Package')}
        **Vulnerability ID:** {vulnerability.get('VulnerabilityID', 'No ID')}
        **Description:** {description}
        **Severity:** {vulnerability.get('Severity', 'Unknown')}
        **Link:** {vulnerability.get('PrimaryURL', 'No link available')}
        """
        create_github_issue(title, body)
        issue_count += 1 

    if issue_count >= max_issues:
        break 

```


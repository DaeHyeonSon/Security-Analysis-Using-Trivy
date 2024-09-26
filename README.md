# Security-Analysis-Using-Trivy
**Trivy**를 활용한 Docker 이미지 & Git Repository **취약점 분석** 

<h2 style="font-size: 25px;"> TEAM 👨‍👨‍👧 <br>
</h2>

|<img src="https://avatars.githubusercontent.com/u/81280628?v=4" width="100" height="100"/>|<img src="https://avatars.githubusercontent.com/u/86951396?v=4" width="100" height="100"/>
|:-:|:-:|
|[@손대현](https://github.com/DaeHyeonSon)|[@이아영](https://github.com/ayleeee)|
---

### 개요 🚩
취약점 스캐너로써 잘 알려진 **Trivy**를 활용하여 **프로그램 배포 시 취약점**을 **진단**하며 이를 **분석**해보고자 한다. 

### 구조 ⚙
Trivy의 구조는 다음과 같다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/99410af8-3056-4512-8c1d-b80c7c4ad161" width="50%">
</div>

### 취약점 진단 툴 사용 이유 🙄

<details>
<summary><i> 왜 Trivy?</i> </summary>
<div markdown="1">

Trivy는 오픈 소스 커뮤니티에서 널리 사용되며 최신 취약점 정보와 개선 사항이 신속하게 반영된다. 이러한 점에서 Trivy는 대중적인 도구라고 생각한다. 또한, Docker 이미지, 파일 시스템, Git 리포지토리 등 다양한 소스에서 취약점을 스캔할 수 있으며, Kubernetes와 같은 클라우드 네이티브 환경에 원활하게 통합되어 넓은 확장성을 갖추고 있어 선택하게 되었다.

</div>
</details>


컨테이너는 이미지를 기반으로 생성되며, 이러한 이미지에는 다양한 취약점이 존재할 수 있다. 배포 전에 이러한 취약점을 사전 발견하면 보안 사고를 예방하고, 문제를 조기에 해결하여 미래에 발생할 수 있는 큰 비용이 수반되는 보안 사고를 방지할 수 있다. 이러한 선제적 대응은 전반적인 시스템 보안을 강화하고, 운영의 안정성을 높이는 데 기여한다.

### 특징 ⚡

- <b>다양한 스캔 대상 지원</b>: 컨테이너 이미지, 파일 시스템, 코드 리포지토리, IaC 파일 등 다양한 대상을 스캔할 수 있다.
  
- <b>빠른 스캔 속도</b>: Trivy는 캐싱 메커니즘을 통해 스캔 속도를 향상시킨다.
  
- <b>광범위한 취약점 데이터베이스</b>: CVE, NVD 등 여러 데이터베이스와 연동하여 최신 취약점을 탐지한다.
  
- <b>간편한 통합</b>: GitLab CI/CD 파이프라인에 쉽게 통합할 수 있는 유연한 설정 및 옵션을 제공한다.
  
- <b>다양한 출력 형식</b>: JSON, 테이블 등 다양한 형식으로 스캔 결과를 출력하여 후속 처리가 용이하다.
  
- <b>플러그인 아키텍처</b>: 필요에 따라 기능을 확장할 수 있는 플러그인 지원한다.


### 스캔 가능 범위 🌐

- Container 이미지
- 파일 시스템
- git repository
- VM 이미지
- Kubernetes
- AWS

## Trivy 실습 🔥

### Docker 이미지 취약점 진단

<br>

**[1] NGINX 취약 버전을 통한 실습** <br><br>
먼저 Trivy 설치를 진행한다.

```bash
sudo apt-get update
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy
```

취약점이 포함된 Nginx Docker 이미지를 생성한다.

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

※ `Alpine 3.12` 리포지토리에 해당 버전이 존재하지 않아 `run`단계에서 오류가 발생 가능하다 -> 버전을 맞춰주면 해결 가능하다.
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
<br><br>
앞서 진행했던 실습을 바탕으로 [WOOSO](https://github.com/DaeHyeonSon/WhiteClothesPeople.git)의 취약점을 파악해보았다. 

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
2. Trivy로 이미지 취약점 스캔 후 문서화<br>
  > 나온 결과값을 vulnerabilites.json 파일로 내보낸다.
```cmd
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/output aquasec/trivy:latest image --format json -o /output/vulnerabilities.json wooso

```
3. 결과값을 모두가 볼 수 있도록 issue 탭에 등록<br>
  > 현재는 일부만 등록되도록 설정하였다. 
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
<br>

**[3] GIT Repository 취약점 진단**

프로젝트로 진행한 Git Repository의 보안 취약점을 진단해보고자 한다. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/8ce48572-1487-48c2-bb6f-5c82e8059ac9" width="50%">
</div>

명령어는 간단하다.

```bash
trivy repo https://github.com/DaeHyeonSon/step04_miniProject #git 주소입력
```

진단 결과는 다음과 같다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/4452b477-c291-4b5a-9559-3dd0d6ce3f1c" width="70%">
</div>

<div align="center">
  <img src="https://github.com/user-attachments/assets/2468680d-b9be-4b78-8aa8-dc89cf3b107a" width="70%">
</div>

자세한 취약점 설명은 각각의 링크를 통해 확인 가능하다.

- https://access.redhat.com/security/cve/cve-2020-36518
- https://access.redhat.com/security/cve/CVE-2021-46877
- https://access.redhat.com/security/cve/CVE-2022-42003
- https://access.redhat.com/security/cve/CVE-2022-42004


### **결론** ✅<br>
본 프로젝트에서는 **Trivy**를 활용하여 **Docker 이미지의 취약점**을 진단하고, 이를 **JSON 형식의 보고서로 작성**함으로써 보안 문제를 효율적으로 파악하고 해결할 수 있는 기반을 마련하였다. 이러한 **진단 및 보고 프로세스를 자동화하면**, 이미지 업데이트가 발생할 때마다 스캔 프로세스와 GitHub 이슈 생성을 원활하게 수행할 수 있을 것이다. 이는 **워크플로우를 최적화**하고, 발생하는 취약점에 **신속하게 대응**함으로써 애플리케이션의 보안 상태를 강화할 수 있을 것으로 사료된다.

<hr>

## Reference 🧷
https://aquasecurity.github.io/trivy/v0.18.3/ <br>
https://betterprogramming.pub/static-analysis-of-container-images-with-trivy-8d297c4f1dd3 <br>
https://faun.pub/how-to-scan-docker-images-e08a7b909ea0

*본 실습은 위 자료를 참조하여 제작하였습니다.*

수정

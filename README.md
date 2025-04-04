# Grafanauts (Grafana  + Astronauts 🧑‍🚀)

<br>

## 📍 Outline
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Purpose](#2%EF%B8%8F⃣-purpose)
- [3️⃣ Contents](#3%EF%B8%8F⃣-contents)
- [4️⃣ Conclusion](#7%EF%B8%8F⃣-conclusion)

<br><br>

## 1️⃣ Contributors
<br>

| <img src="https://avatars.githubusercontent.com/u/82265395?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/193213283?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/114290855?v=4" width="200px"> | 
| :---: | :---: | :---: |
| [구민지](https://github.com/minjee83) | [박진현](https://github.com/jinhyunpark929) | [이성빈](https://github.com/andytjdqls) |

<br>

## 2️⃣ Purpose

이 프로젝트는 **Spring Boot 애플리케이션**과 **MySQL 서버**에 대해 Prometheus & Grafana를 활용한 **모니터링 환경을 구축**하고,  
**스트레스 테스트 및 성능 관측**을 통해 실시간 데이터를 수집하고 시각화하는 것을 목표로 합니다.

<br>

# 📌 프로젝트 개요
  
- **기술 스택:**  <br>
![Prometheus](https://img.shields.io/badge/Prometheus-000000?logo=prometheus&logoColor=orange&labelColor=white) ![Grafana](https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white)![mysqld_exporter](https://img.shields.io/badge/mysqld_exporter-4479A1?logo=mysql&logoColor=white)![node_exporter](https://img.shields.io/badge/node_exporter-539E43?logo=gnu&logoColor=white)![Spring Boot](https://img.shields.io/badge/SpringBoot-6DB33F?logo=springboot&logoColor=white)![Java](https://img.shields.io/badge/Java-007396?logo=java&logoColor=white)![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?logo=virtualbox&logoColor=white)![VMware Workstation](https://img.shields.io/badge/VMware-607078?logo=vmware&logoColor=white)![STS](https://img.shields.io/badge/Spring_Tools_SUITE-6DB33F?logo=spring&logoColor=white)![Mobaxterm](https://img.shields.io/badge/Mobaxterm-333333?logo=windows&logoColor=white)![Slack](https://img.shields.io/badge/Slack-4A154B?logo=slack&logoColor=white)

- **모니터링 구성요소:**
  - `node_exporter`: 시스템 리소스(CPU, Memory, Network 등)
  - `mysqld_exporter`: MySQL 상태 및 쿼리 메트릭
  - `Prometheus`: 메트릭 수집기
  - `Grafana`: 시각화 도구
- **부하 테스트 도구:**
  - `stress`: CPU 기반 부하 생성
  - 'stree ng' : 
  - `sysbench`: MySQL 트랜잭션 부하 테스트

<br>

## 3️⃣ Contents

<br>

## ⚙️ 모니터링 구성 및 테스트 흐름

### 1️⃣ Node Exporter 관측 (시스템 리소스)

- `node_exporter`를 통해 CPU, 메모리, 디스크, 네트워크 등의 **시스템 자원 상태**를 Prometheus에 수집
- Grafana에서 **Dashboard ID: `1860`** 불러와 실시간 시각화 확인

```bash
# node_exporter 실행 예시
./node_exporter &
```

#### 주요 PromQL 예시

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)   # CPU 사용률
node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes                      # 메모리 사용량
rate(node_network_receive_bytes_total[1m])                                       # 네트워크 수신량
```

---

### 2️⃣ Spring Boot App 실행 및 관측

- JDK 설치 후, `Spring Boot` 애플리케이션을 직접 실행하여 시스템 부하 유도
- `kubectl port-forward`와 `curl`로 정상 동작 확인

```bash
java -jar myapp.jar
kubectl port-forward service/myapp-service 8087:8082
curl localhost:8087
```

- 애플리케이션 부하로 인한 **CPU/메모리 변화**를 node_exporter로 실시간 관측

---

### 3️⃣ 부하 테스트 도구 활용

#### 📌 App 부하 테스트: `stress` 라이브러리

```bash
sudo apt install stress
stress --cpu 4 --timeout 60
```

→ CPU 사용률 급등을 Grafana로 실시간 관측

---

#### 📌 MySQL 부하 테스트: `sysbench`

```bash
sudo apt install sysbench

# 준비 단계
sysbench /usr/share/sysbench/oltp_read_write.lua \
  --mysql-host=localhost --mysql-user=testuser \
  --mysql-password=testpass --mysql-db=testdb prepare

# 부하 실행
sysbench /usr/share/sysbench/oltp_read_write.lua \
  --mysql-host=localhost --mysql-user=testuser \
  --mysql-password=testpass --mysql-db=testdb \
  --threads=8 --time=60 --report-interval=10 run
```

---

### 4️⃣ MySQL Exporter 관측

- `mysqld_exporter`를 통해 MySQL 내부 상태 (쿼리 수, 커넥션, 슬로우 쿼리 등) 수집
- Prometheus에서 scrape 후 Grafana 대시보드에 시각화

#### 서비스 유닛 파일 생성

```bash
sudo tee /etc/systemd/system/mysqld_exporter.service <<EOF
[Unit]
Description=MySQL Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=mysql
Group=mysql
Type=simple
ExecStart=/usr/local/bin/mysqld_exporter \\
  --config.my-cnf=/etc/.my.cnf \\
  --web.listen-address=:9104
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

```bash
# systemd 등록 및 자동 실행
sudo systemctl daemon-reload
sudo systemctl enable mysqld_exporter
```

---

## 🔧 Trouble Shooting

### ⚠️ mysqld_exporter 권한 오류

```bash
/usr/local/bin/mysqld_exporter --config.my-cnf=/etc/.my.cnf --web.listen-address=:9104
# 오류:
failed to load config from /etc/.my.cnf: open /etc/.my.cnf: permission denied
```

#### ✅ 해결 방법:

```bash
sudo chown prometheus:prometheus /etc/.my.cnf
sudo chmod 600 /etc/.my.cnf
```

---

### 🔁 재부팅 후 서비스 자동 실행 확인

```bash
sudo init 6    # 재부팅
sudo systemctl status mysqld_exporter
```

#### 💡 효과:
- 서버 재시작 후에도 `mysqld_exporter` 자동 실행
- Prometheus에서 계속 scrape 가능
- Grafana 대시보드 연결 유지

---

## 7️⃣ Conclusion

본 프로젝트에서는 Prometheus 및 Grafana를 활용하여 Spring Boot 애플리케이션과 MySQL 서버의 실시간 모니터링 환경을 구성하였습니다.  
`node_exporter`, `mysqld_exporter`를 통해 시스템 및 데이터베이스 리소스 사용 현황을 수집하고,  
`stress` 및 `sysbench`를 활용한 부하 테스트로 각 exporter의 반응을 관찰하며 모니터링 대시보드를 실시간으로 확인하였습니다.

특히 재부팅 후에도 exporter가 자동 실행되고 데이터 연속성이 유지되도록 systemd 서비스를 구성함으로써,  
실제 운영 환경에서도 사용할 수 있는 모니터링 기초 역량을 확보할 수 있었습니다.

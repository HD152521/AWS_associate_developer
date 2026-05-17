# Day 49 - CloudWatch 고급: Dashboards, Container Insights

📅 날짜: 2026년 7월 22일 (수요일)  
🎯 주제: CloudWatch 고급 기능  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- CloudWatch Dashboards로 통합 모니터링 화면을 구성한다
- Container Insights로 ECS/EKS 컨테이너를 모니터링한다
- CloudWatch Synthetics로 엔드포인트를 지속적으로 테스트한다

---

## 📖 이론 내용

### 1. CloudWatch Dashboards

여러 서비스의 지표를 하나의 화면에서 모니터링합니다:

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# 대시보드 생성
dashboard_body = {
    "widgets": [
        {
            "type": "metric",
            "properties": {
                "title": "EC2 CPU 사용률",
                "metrics": [
                    ["AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890"]
                ],
                "period": 300,
                "stat": "Average"
            }
        },
        {
            "type": "metric",
            "properties": {
                "title": "Lambda 오류 수",
                "metrics": [
                    ["AWS/Lambda", "Errors", "FunctionName", "my-function"]
                ],
                "period": 60,
                "stat": "Sum"
            }
        },
        {
            "type": "alarm",
            "properties": {
                "title": "알람 상태",
                "alarms": ["arn:aws:cloudwatch:..."]
            }
        }
    ]
}

cloudwatch.put_dashboard(
    DashboardName='MyAppDashboard',
    DashboardBody=str(dashboard_body)
)
```

### 2. CloudWatch Logs Metric Filter

로그에서 지표를 생성합니다:

```bash
# 로그에서 ERROR 패턴 감지 → 지표 생성
aws logs put-metric-filter \
    --log-group-name "/aws/lambda/my-function" \
    --filter-name "ErrorCount" \
    --filter-pattern "ERROR" \
    --metric-transformations \
        metricName=ErrorCount,metricNamespace=MyApp,metricValue=1

# 생성된 지표로 알람 설정
aws cloudwatch put-metric-alarm \
    --alarm-name "LambdaErrors" \
    --metric-name "ErrorCount" \
    --namespace "MyApp" \
    --period 60 \
    --evaluation-periods 1 \
    --threshold 5 \
    --comparison-operator GreaterThanThreshold
```

### 3. CloudWatch Container Insights

ECS/EKS/Kubernetes의 컨테이너 레벨 모니터링:

```bash
# ECS에 Container Insights 활성화
aws ecs update-cluster-settings \
    --cluster my-cluster \
    --settings name=containerInsights,value=enabled
```

**모니터링 지표:**
- CPU/메모리 사용률 (컨테이너 레벨)
- 네트워크 트래픽
- 디스크 I/O
- 태스크/서비스 상태

### 4. CloudWatch Synthetics

실제 사용자처럼 엔드포인트를 주기적으로 테스트합니다:

```javascript
// Canary 스크립트 예시
const synthetics = require('Synthetics');
const syntheticsConfiguration = synthetics.getConfiguration();

const apiCanaryBlueprint = async function () {
    // API 엔드포인트 테스트
    const requestOptions = {
        hostname: 'api.myapp.com',
        method: 'GET',
        path: '/health',
        port: 443,
        protocol: 'https:'
    };
    
    const response = await synthetics.executeHttpStep('Health Check', requestOptions);
    
    // 응답 검증
    if (response.statusCode !== 200) {
        throw new Error(`상태 확인 실패: ${response.statusCode}`);
    }
};

exports.handler = async () => {
    return await apiCanaryBlueprint();
};
```

**Synthetics 사용 케이스:**
- API 가용성 모니터링 (주기적 HTTP 요청)
- 사용자 흐름 테스트 (로그인, 구매 등)
- 다중 리전에서 동시 테스트

### 5. CloudWatch Anomaly Detection

머신 러닝 기반으로 비정상적인 지표 패턴을 자동 감지합니다:

```bash
# Anomaly Detection 활성화
aws cloudwatch put-anomaly-detector \
    --namespace AWS/Lambda \
    --metric-name Duration \
    --dimensions Name=FunctionName,Value=my-function \
    --stat Average
```

---

## 아키텍처 다이어그램

```
통합 모니터링 대시보드 아키텍처
================================

[애플리케이션 스택]
  EC2 / Lambda / ECS / RDS
          |
          v
[CloudWatch]
  지표 → Dashboards (시각화)
  로그 → Metric Filter → 지표 → 알람
  Container Insights → 컨테이너 지표

CloudWatch Synthetics 모니터링
================================

매 5분마다:
[Synthetics Canary]
     |
     | 실제 HTTP 요청
     v
[API 엔드포인트]
     |
  200 OK ← 정상
  5xx ← [CloudWatch 알람] → [SNS 알림]
```

---

## ⭐ 핵심 포인트

1. ⭐ **Logs Metric Filter**: 로그 패턴에서 지표 생성, 알람 연동
2. ⭐ **Container Insights**: ECS/EKS 컨테이너 레벨 모니터링
3. ⭐ **Synthetics**: 주기적 엔드포인트 테스트, 실제 사용자 시뮬레이션
4. ⭐ **Anomaly Detection**: ML 기반 비정상 패턴 자동 감지
5. ⭐ **Dashboards**: 다중 서비스 통합 모니터링, 교차 계정 가능

---

## 📝 연습 문제

**문제 1.** Lambda 로그에서 ERROR 문자열 발생 시 알람을 설정하려면?

A) CloudTrail 알람  
B) CloudWatch Logs Metric Filter로 지표 생성 후 알람  
C) X-Ray 알람  
D) EventBridge 직접 규칙  

**정답: B** - Logs Metric Filter로 ERROR 패턴을 감지하여 지표로 변환하고, 해당 지표에 알람을 설정합니다.

---

**문제 2.** ECS 컨테이너의 CPU 사용률을 모니터링하려면?

A) EC2 기본 지표 사용  
B) Container Insights 활성화  
C) CloudWatch Agent 설치  
D) X-Ray 활성화  

**정답: B** - Container Insights를 활성화하면 컨테이너 레벨의 CPU, 메모리, 네트워크 지표를 수집할 수 있습니다.

---

**문제 3.** API 엔드포인트가 24시간 정상 동작하는지 지속적으로 테스트하려면?

A) CloudWatch Alarms  
B) CloudWatch Synthetics  
C. Health Check  
D) Route 53 Health Check  

**정답: B** - CloudWatch Synthetics는 주기적으로 스크립트를 실행하여 엔드포인트의 가용성과 기능을 자동으로 테스트합니다.

---

**문제 4.** CloudWatch Anomaly Detection의 역할은?

A) 수동으로 임계값 설정  
B) ML로 지표 패턴을 학습하여 비정상 패턴 자동 감지  
C) 보안 위협 분석  
D) 비용 분석  

**정답: B** - Anomaly Detection은 머신 러닝으로 지표의 정상 패턴을 학습하고 이를 벗어나는 비정상적인 값을 감지합니다.

---

**문제 5.** 여러 AWS 계정의 지표를 하나의 대시보드에서 모니터링하려면?

A) 각 계정에서 별도 대시보드 생성  
B) CloudWatch 교차 계정 대시보드  
C) 불가능  
D) AWS Organizations 필요  

**정답: B** - CloudWatch는 교차 계정(Cross-Account) 대시보드를 지원하여 여러 계정의 지표를 하나의 대시보드에서 볼 수 있습니다.

---

## 📌 오늘의 요약

1. CloudWatch Dashboards: 다중 서비스 통합 모니터링, 교차 계정 지원
2. Logs Metric Filter: 로그 패턴 → 지표 → 알람으로 연계
3. Container Insights: ECS/EKS 컨테이너 레벨 지표 수집
4. Synthetics: 주기적 엔드포인트 테스트, 실제 사용자 시뮬레이션
5. Anomaly Detection: ML 기반 비정상 패턴 자동 감지

---
title: Vercel·AWS 계정 및 결제 방식 정리
author: lsg
category: 06_참고자료
status: final
created: 2026-07-03
updated: 2026-07-03
tags:
  - Vercel
  - AWS
  - 결제
  - 참고자료
summary_target: false
---

# Vercel · AWS 계정 및 결제 방식 정리
> 작성일: 2026-07-03 | 작성자: 이슬기
> 2026-07-06(월) PM/PD 오프라인 결제 방식 협의를 위한 사전 조사 자료

---

## 목차
1. [Vercel 계정 관련](#1-vercel-계정-관련)
2. [AWS 계정 관련](#2-aws-계정-관련)
3. [Vercel 결제 관련](#3-vercel-결제-관련)
4. [AWS 결제 관련](#4-aws-결제-관련)
5. [비교 요약](#5-비교-요약)

---

## 1. Vercel 계정 관련

### 프로젝트 단위가 아니라 팀(Team) 단위
Vercel은 계정 구조상 **팀 안에 여러 프로젝트를 생성**하는 방식이며, 사용량·청구는 프로젝트별이 아니라 팀 단위로 통합된다.

- 근거: [Spend Management](https://vercel.com/docs/spend-management) — "The spend amount that you set covers metered resources... for all projects on your team."

### 좌석(Seat) 구조
- Pro 플랜은 배포 권한이 있는 팀원 1명당 월 $20이며, 여기에 $20 상당의 사용량 크레딧이 포함된다.
- Owner/Member 좌석은 유료, Viewer 좌석은 무료·무제한.
- 근거: [Vercel Pro Plan (공식 문서)](https://vercel.com/docs/plans/pro-plan)

---

## 2. AWS 계정 관련

### 결제 수단 관리 콘솔
- AWS Billing and Cost Management 콘솔의 **Payment preferences** 페이지에서 결제 수단을 등록·조회·삭제하고 기본 결제 수단을 지정한다.
- 지원 결제 수단: 신용/직불카드 외에 계좌 직접 인출(미국 ACH, 유럽 SEPA)도 가능.
- 여러 AWS 서비스 판매자(SOR, Seller of Record)로부터 인보이스를 받는 경우, 판매자별로 다른 결제 수단을 지정하는 "Payment profiles" 기능도 있음.
- 근거: [Customizing your AWS payment preferences (AWS 공식 문서)](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-payment-method.html)

---

## 3. Vercel 결제 관련

### 3-1. 배포 전에도 결제 수단 등록 가능
결제 수단 등록은 프로젝트 배포 여부와 무관하다. 팀 대시보드 → Settings → Billing → Payment Method에서 언제든 카드 등록 가능.

- 신규 구독은 **24시간 이내**, 갱신 결제는 **14일 이내**에 결제가 성공해야 하며, 실패 시 배포가 모두 중단된다.
- 결제 수단이 없거나 연체(overdue) 상태가 되면: 새 프로젝트 생성 불가, 팀원 추가 불가, 기존 프로젝트 재배포 불가.
- 근거: [Billing FAQ for Pro Plan (공식 문서)](https://vercel.com/docs/plans/pro-plan/billing)

### 3-2. 청구 구조 (선불 크레딧 + 온디맨드 후불)
- 매 빌링 사이클 시작 시, (a) 다음 30일치 좌석·애드온 요금 + (b) 지난 30일간 발생한 초과 사용량을 함께 청구한다.
- 기본 포함량(1TB Fast Data Transfer, 1천만 Edge Requests 등)을 초과하면 좌석 요금에 포함된 크레딧에서 먼저 차감되고, 크레딧 소진 후에는 온디맨드로 추가 청구된다.
- 월 단위 결제만 지원(연납 불가). 연납은 Enterprise 플랜에서만 가능.
- 근거: [Billing FAQ for Pro Plan (공식 문서)](https://vercel.com/docs/plans/pro-plan/billing), [Pricing on Vercel (공식 문서)](https://vercel.com/docs/pricing)

### 3-3. 지출 한도(Spend Limit) — 팀 단위만 지원, 프로젝트별 불가
- Spend Management는 **팀 전체의 사용량 합계**를 기준으로 한도(USD)를 설정하는 기능이며, 개별 프로젝트마다 한도를 따로 거는 기능은 없다.
- 설정 경로: 팀 대시보드 → Settings → Billing → Spend Management 토글 → 한도 금액 입력
- 한도 도달 시 선택 가능한 동작:
  1. 알림 (50% / 75% / 100% 도달 시 웹·이메일, 100%는 SMS도 가능)
  2. Webhook 트리거 (Slack 알림, 정적 페이지 전환 등)
  3. **팀 내 모든 프로젝트의 프로덕션 배포 일괄 정지** (개별 프로젝트 단위 정지 아님)
- 유의점: 사용량 체크가 몇 분 간격 폴링 방식이라 실시간 차단이 아니며, 한도 초과 후에도 몇 분간 사용량이 더 쌓일 수 있다. 정지된 프로젝트는 한도를 올려도 자동 재개되지 않고 개별 수동 재개가 필요하다.
- 근거: [Spend Management (공식 문서)](https://vercel.com/docs/spend-management)

### 3-4. 프로젝트 단위로 과금되는 예외 항목
- 일부 애드온은 프로젝트 단위로 별도 과금된다. 예: Speed Insights는 Pro 플랜 기준 프로젝트당 월 $10 기본요금.
- 반면 Web Analytics는 프로젝트별이 아니라 팀 전체에서 수집된 이벤트 수 기준으로 과금된다.
- 근거: [Limits and Pricing for Speed Insights (공식 문서)](https://vercel.com/docs/speed-insights/limits-and-pricing), [Pricing for Web Analytics (공식 문서)](https://vercel.com/docs/analytics/limits-and-pricing)

---

## 4. AWS 결제 관련

### 4-1. 후불(Postpaid) 청구 모델
- AWS는 순수 후불제. 한 달 사용량이 마감되면 인보이스가 생성되고, 그 시점에 등록된 결제 수단으로 자동 청구된다.
- 마감되지 않은 당월 청구 기간에도 Bills 페이지에서 지금까지 계측된 사용량 기준 예상 청구액을 확인할 수 있다.
- 근거: [A Quick Overview of AWS Payment Methods (AWS 공식 블로그)](https://aws.amazon.com/blogs/aws-cloud-financial-management/a-quick-overview-of-aws-payment-methods/)

### 4-2. 청구·결제 시점
- 기본 결제 수단은 보통 매월 3~5일경 청구되며, AWS는 인보이스 마감일 또는 그 다음 날 결제를 시작한다.
- 즉 **당월 사용량을 다음 달 초에 후불로 청구**하는 구조.
- 근거: [Making payments (AWS 공식 문서)](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-making-a-payment.html)

### 4-3. 지출 한도 관련 참고
- AWS는 Vercel처럼 팀/프로젝트 단위 하드캡(자동 사용 중단) 기능이 기본으로 강하게 제공되지 않는다. 예산 알림(AWS Budgets) 중심의 모니터링이 기본이며, 자동 차단을 원하면 별도 자동화(Lambda + Budgets Action 등) 구성이 필요하다.
- 사용량 규모 산정은 BE에서 별도 논의 예정.

---

## 5. 비교 요약

| 항목 | Vercel | AWS |
|---|---|---|
| 청구 방식 | 선불성 크레딧 + 초과분 온디맨드, 팀 단위 통합 청구 | 순수 후불(postpaid), 월 사용량 마감 후 청구 |
| 결제 수단 등록 시점 | 배포 이전에도 가능 (오히려 사실상 필수 선행 조건) | 계정 생성 시 등록, 콘솔에서 언제든 변경 가능 |
| 지출 한도 설정 단위 | 팀 단위만 가능 (프로젝트별 불가) | 기본 하드캡 없음, 예산 알림(Budgets) 중심 |
| 결제 주기 | 매월(연납 불가, Enterprise 제외) | 매월 (사용량 마감 후 후불) |

---

## 참고 링크 (근거 자료)
- [Pricing on Vercel](https://vercel.com/docs/pricing)
- [Vercel Pro Plan](https://vercel.com/docs/plans/pro-plan)
- [Billing FAQ for Pro Plan](https://vercel.com/docs/plans/pro-plan/billing)
- [Understanding Vercel's Pro Plan Trial](https://vercel.com/docs/plans/pro-plan/trials)
- [Spend Management](https://vercel.com/docs/spend-management)
- [Limits and Pricing for Speed Insights](https://vercel.com/docs/speed-insights/limits-and-pricing)
- [Pricing for Web Analytics](https://vercel.com/docs/analytics/limits-and-pricing)
- [Customizing your AWS payment preferences](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-payment-method.html)
- [A Quick Overview of AWS Payment Methods](https://aws.amazon.com/blogs/aws-cloud-financial-management/a-quick-overview-of-aws-payment-methods/)
- [Making payments](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/manage-making-a-payment.html)

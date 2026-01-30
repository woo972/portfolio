## Context & Constraints
시그널링 서버는 유저(SDK client)와 AI worker를 Websocket 연결을 통해 실시간 통신할 수 있게 하는 핵심 컴포넌트이다. 채팅을 위한 텍스트 전달 뿐 아니라 비디오 송출 트리거, 음성 메시지 저장, 각종 매트릭 관리 등 상호작용의 모든 라이프사이클을 제어한다.

매 서버 재배포마다 모든 Websocket session은 종료되며, 아래 문제가 발생한다.
- 채팅과 미디어 송출이 즉시 종료된다.
- SDK client는 채팅 히스토리를 잃어버린다. 
- 새로운 서버로 재접속을 위한 최소한의 시간을 추가로 소비(약 20초)해야한다.    
- 고객 경험 저하를 방지하기 위해 새벽시간대 배포 수행이 강제된다.     
### 핵심 제한사항
1. Websocket 기반, Stateful connection.
2. 시그널링 서버와 코어 서버(SDK, 미디어 등 서비스의 모든 데이터 및 설정 정보 제공)간의 강결합.
3. 유저가 신규 AI Worker와 매칭되면, 전송되된 미디어가 끊기기 때문에 Old Server에서 연결되었던 AI Worker와 New Server에서도 연결되어야 함.
4. 롤링업데이트 상황에서 배포로 인해 클라이언트 재접속 시도시 Reconnect storm 야기.

목표는 유저 경험에 악영향을 주지 않으면서 언제나 배포 가능한 시스템을 만드는 것이다.
## Design Options Considered
### Option 1: Client Dual-Connection Strategy
각 클라이언트가 기존 서버와 신규 서버에 대한 커넥션을 관리한다.
- Pros: 개념이 간단하고 서버 구현이 쉬움.
- Cons: 단순히 연결만 하면 되는게 아니고 기존 매칭 상대방을 찾아야하기 때문에 채택 불가능.
### Option 2: Stateless Reconnect with Session Recreation
연결 끊김을 허용하고 배포 후 세션을 재연결한다.
- Pros: 아키텍쳐 변경 최소화.
- Cons: 유저 끊김 발생으로 채택 불가능.
### Option 3 (선택): Recovery Token–Based Session Handover
Old 서버에서 New 서버로 명시적인 session handover 메커니즘을 적용한다.
- Pros: 가장 확실한 해결책.
- Cons: 구현 복잡하고 재접속 로직으로 컴포넌트 간 강결합(시그널링 서버, 클라이언트, 인프라).
## Decision 
블루그린 배포로 인프라레벨 변경을 적용한 뒤 recovery token 기반 reconnection 및 handover architecture를 구현하였다. 
### 핵심 결정
1. Recovery token을 통한 명시적인 세션 신원 확인.
    - initial connection 시 발행.
    - session id, 연결된 session id 등 재연결에 필요한 필수 정보 저장.
    - 세션 종료시 폐기.
2. AI Worker-first 재연결 정책.
    - Worker는 SDK 클라이언트 접속 전에 재연결 후 대기.
    - SDK 클라이언가 재매칭이 불가능한 Worker와 연결되지 않게 처리.    
3. 시그널링 서버가 중심이 되는 Graceful shutdown.
    - K8S에 의해 Old pod가 shutdown period들어가며 세션 재연결 스테이지로 진입.
    - 명시적인 handover signal을 통해 세션 이동을 지원.
4. Infrastructure-level 트래픽 분리.
    - Argo Rollout을 통해 Blue-Green 배포 도입.
    - 재 연결은 New pod 그룹으로 한 번만 발생.

## Finalized Architecture
![websocket_zero_downtime_deployment_sequence](./sequence.png)
## **Trade-Offs & Risks**
- 프로토콜 및 시퀀스 복잡도 증가.
- 문제 발생 시 크로스 도메인 협업 필요 (Backend, SDK, Worker, DevOps).
- 추가적인 상태 관리 필요 (Recovery token).
- Token이 적절하게 비활성화되지 않는 경우 오용 가능성.
- 재연결 로직에서 부분적인 실패 가능성.
## **Outcome**
해당 프로젝트는 사전에 지정한 수치를 달성하며 종료됨.
- Input: reconnect 시나리오 테스트에서 Critical Path 100% 통과, Non Critical Path 90% 이상 통과
- Output: 재접속 성공률 90% 이상 (배포간 세션 로스 비율 10% 미만)    
운영상으로는 팀이 “배포는 항상 주의하면서 하는 것”에서 “언제든 자신있게 하는 것”으로 전환됨.
## **What I’d Change Next Time**
- 세션별 메트릭을 고도화하여 라이프사이클 추척 용이성 확보.
- 자동화된 재연결 카오스 테스팅 환경 구축을 선행하여 E2E테스트를 강화.
- 타 사 사례 참고하여 더 유연하고 일반화된 방법 적용

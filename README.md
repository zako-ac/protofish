![banner](/assets/protofish-banner.png)
> Dedicated to the one who is an author of the name *Protofish*.

# Protofish
> ✨🎀 Tragbare Stream Transportation 🎏
다중화된 바이너리 스트림을 위한 추상 전송 프로토콜

## 왜 Protofish를 쓰나요?
Protofish가 고안되기 전까지 TapHub는 전통적인 HTTP/WS 방식을 사용해 오디오 스트림을 전송했습니다. 이는 가장 쉽고 간단한 방법입니다. 그러나, 이 방법은 금새 여러 문제점들의 베일을 벗겨냈습니다. 다음 챕터는 이전 방식의 주요 문제점과 Protofish가 그들을 해결한 방법에 관한 간략한 설명입니다.

## 명세서
[여기 있습니다](protofish/protofish.md). 모든 문서에 번역, 문장 다듬기, 기술적 수정 등의 기여는 언제나 환영입니다.

## Protofish가 해결하는 문제
### TCP Head-of-Line 문제
![hol](/assets/tcp-hol.png)
TapHub가 사용하는 HTTP/1.1은 TCP 기반이기에, TCP의 HOL 문제를 수반합니다. TCP는 패킷의 무결성과 패킷 순서의 일관성을 보장합니다. 따라서 잘못된 패킷(그림에서 6번 패킷)이 들어온다면 그 패킷이 복구될 때까지 전체 스트림은 중단됩니다.\
Protofish는 UDP를 통해 이 문제를 해결합니다. Zako2 Infrastructure에서 사용되는 표준 Protofish 업스트림 구현체인 QUICfish는 UDP기반 프로토콜인 QUIC이 제공하는 비신뢰 스트림(Unreliable Stream)을 이용합니다. 이는 전통적인 UDP기반 손실 오디오 전송 시스템에 QUIC의 강력한 보안 및 안정성을 추가합니다.

### HTTP의 일회성
![taphub-principle](/assets/taphub-principle.png)
TapHub는 최소한 2개의 응답 패킷을 필요로 합니다. TapHub가 Tap에게 오디오 요청을 보냈을 때, 자세한 내용을 생략한다면 다음과 같습니다.
- **ACK** 해당 Tap이 온라인이고 음원을 생성할 수 있는 상태임을 표시
- **RES** 실제 음원 데이터
HTTP 기반의 응답에서 TapHub는 HTTP 소켓의 시작을 **ACK**, 그것의 데이터는 **RES**로 판단합니다. 그러나 이 방법은 여러 문제가 존재합니다.
- 통상적이지 않은 HTTP의 기능을 중요 로직에 이용합니다.
- 저수준 HTTP 클라이언트가 필요합니다.
- 추가적인 패킷 확장이 어렵습니다.
Protofish는 Context 시스템을 통해 이 문제를 해결합니다. 프로토콜의 채널을 통해 한 개의 요청에 대한 여러 번의 대화 세션을 효율적으로 추적할 수 있습니다.

### All-in-One
이 모든 기능이 하나의 프로토콜에 담겨 있습니다. 한 개의 논리적 연결만으로 안정적인 Tap 연결의 유지가 가능합니다.

## Protofish, QUICfish, Zakofish
Protofish, QUICfish, Zakofish는 모두 다른 개념입니다.

### Protofish
Protofish는 바이너리 스트림과 데이터를 효율적으로 전송하기 위한 프로토콜입니다. 실시간 오디오나 Zako2에 관한 직접적인 연관은 존재하지 않습니다. 또한, 추상적인 작동 방식의 설명이기에 Protofish 명세만으론 IP기반 네트워크 위에서 동작할 수 없습니다.

### QUICfish
QUICfish는 Protofish를 콘크리트한 QUIC 프로토콜 위에 구현한 존재입니다. QUICfish는 직접적으로 IP 네트워크 위에서 통신할 수 있습니다.

### Zakofish
Zakofish는 Zako2 Infrastructure의 전통적인 HTTP/WS 시스템을 보완하기 위해 QUICfish 위에서 동작하도록 설계된 통신 프로토콜입니다. Zakofish는 HTTP기반 Zako2 Tap 통신을 완전히 대체합니다. (단, 하위 호환성을 위해 HTTP/WS 시스템은 보존됩니다.)

## 기여하기
기술 구현은 물론 번역, 문장 재구성 등을 포함한 모든 종류의 기여는 언제나 환영입니다.

### 주의 사항
- 문서의 짜임 형태를 보존해주세요.
- 이해하기 쉽게 작성해주세요.

## 결론
Protofish는 Zako2 Infrastructure를 최적화하기 위해 고안되었지만, 실시간 스트림 전송에 특화된 다양한 분야에 사용 가능하도록 최적화되었습니다.\
그리고 자코는 귀엽습니다.

## Credits
- MincoMK (@minco_rl)
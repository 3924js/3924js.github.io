 * 언리얼
	 * 프로젝트
	 * 레벨
	 * 블루프린트(Blueprint)
		 * 언리얼에서 제공하는 비주얼 스크립팅(Visual scripting) 언어, 노드들의 연결로 구성
	 * 이벤트(Event)
		 * 사건이 발생했을때 호출되는 함수들
		 * BeginPlay: 게임 플레이가 시작될 때 호출되는 함수
		 * Tick: 매 프레임마다 호출되는 함수
	 * 함수(Function)
		 * 연산이나 기능을 가지고 작동하는 노드
		 * Print String: 기본적인 언리얼 출력 함수
	 * 변수(Variable)
		 * 변할 수 있는 값을 저장, 언리얼에서 대게 속성(Property)로 지칭
		 * Boolean
		 * 정수
			 * Byte
			 * Integer
			 * Integer64
		 * Float
		 * Name(불변인 문자열)
		 * String
		 * Text(다국어 변환이 되는 문자열)
	 * 변수를 블루프린트의 EventGraph로 드래그하면 Get혹은 Set을 할 수 있음
		 * Get: 변수 값에 대해 한 접근, Read에 해당
		 * Set: 변수 값을 설정, Write에 해당
   * 연산 노드
     * 비교 연산 노드
       * ==
       * !=
       * <
       * \>
       * <=
       * \>=
     * 논리 연산 노드
       * And
       * Or
       * Not
       * Xor
	 * 흐름 제어
		 * Sequence: 주어진 노드들의 순차적인 실행을 제어
		 * Branch:
		 * 반복문
		 * 열거형
     * 구조체
     * Switch
       * 주어진 값에 따라
   * C언어
     * 2-4 비트 연산자(Bit operators)
       * 비트 논리 연산자(Bit logical operators)
         * 비트별로 논리 연산을 수행
           * &(논리합, and)
           * |(논리곱, or)
           * ~(부정, not)
           * ^(베타합, xor)
         * ex) 13 & 9 => 1101(2) & 1001(2) => 1001(2) => 9
       * 비트 이동 연산자(Bit Shift operators)
         * 2진수 기준 칸을 옮기는 연산자
           * \<\<, >>
         * ex) 3 \<\< 2 => 0011(2) \<\< => 1100(2) (왼쪽으로 2칸 이동) => 12
         * 메모리 상에서 위치를 옮겨주는(shift) 효과는, 2로 곱하고 나누는 결과와 동일. 특정 상황에서 곱셈, 나눗셈보다 빨리 작동하기에 최적화에 도움이 됨.
     * 3-1 C언어: 절차지향언어(procedural programming language), 실행순서에 대해 잘 이해할것
     * 3-2 if문(if statement)
       * 조건의 참 거짓 여부에 따라 실행여부가 결정되는 조건문
       * 구조<br>if(condition) {statement;}
       * 정말 간단한 한줄 커맨드의 경우 스코프({})가 없어도 문법적으로 이상 없음.(숏코딩 기법중 하나)
     * 3-3 else
       * if와 함께 쓰여 if의 조건 외에 실행되도록 해주는 조건문
       * 구조<br>if(condition) <br>{statement1;}<br> else <br>{statement2;}
       * 3항 연산자로 대체 가능<br> (condition ? statement1 : statement2)
     * 3-4 else if
       * 3개 이상의 여러 조건에 따른 실행을 보장
       * 구조<br>if(condition) <br>{statement1;}<br> else if(condition) <br>{statement2;}<br> else<br>{statement3;}
       * 단순히 여러개의 if 조건문의 나열은, 조건만 맞다면 모두 실행 가능, if-else if-else의 경우 개중 하나만 실행.
     * 3-5 중첩 if (Nested if statement)
       * if문 안에도 if가 들어가 있을 수 있음.
     * 3-6 Switch
     * 4-1 while
       * 조건이 만족하면 계속해서 작동하는 반복문
       * 구조<br>while (condition)<br>{<br>statement;<br>}
     * 4-2 for
       * 반복자에 기반하여 조건을 만족하는 동안 작동하는 반복문
       * 구조예시 <br> int i = 0; for(i; i\< 10; i++) //i가 10이 될때까지 반복 <br>{<br>statement;<br>}
     * 4-3 do-while
       * while의 조건을 판단하기 전에 명령들을 먼저 실행 한 후 평가하는 while의 변형 (최소 1번 실행 보장)
       * 구조<br>do{<br> statement;<br>}while(condition)
     * 4-4 무한 반복문
       * 무한히 반복되는 반복문, while과 for 2개 모두 구현 가능하나 while을 주로 사용 <br> while(true){<br>statement;<br>}<br><br> int i = 1;<br>for(i; 1; i++) //가운데 조건식이 true이니 무한반복<br>{<br>statement;<br>}
       * 무한히 빠져나오지 못하는 반복문은 작동 못하는 것이나 다름 없어 제어를 필요로 함.
         * continue: 다음 반복으로 바로 실행
           * ex)<br> while(true){<br>statement1; continue; statement2; //contidnue에서 끊기기에 실행되지 않음<br>}
         * break: switch의 break와 동일, 반복문의 종료, 반복문 구조에서의 탈출(break, escape)을 의미
           * ex)<br> while(true){<br>statement1;<br>break;<br>statement2; //break에서 반복문 종료, statement2 실행되지 않음<br>}

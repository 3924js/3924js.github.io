---

title: "C++: 포인터(Pointer) \& 레퍼런스(Reference)"

author: Jaeseong Kim

date: 2026-03-04 12:00:00 +0800

categories: \[C++, C++ Basics]

tags: \[C++, C++ Basics]

---

\* ## 포인터(Pointer)

&nbsp;	\* 메모리상의 주소값을 저장하기 위한 자료형, 모든 자료형은 포인터로 만들 수 있다.

&nbsp;	\* 동적인 메모리 접근과 관리를 가능하게 하고, 큰 객체를 복사하지 않고 주소만 전달할 수 있게 

&nbsp;		```c++

&nbsp;		int main(){

&nbsp;			int val = 5;

&nbsp;			//선언: 포인터자료형\* 이름

&nbsp;			//변수의 주소 접근: \&변수

&nbsp;			int\* ptr = \&val; //val의 주소를 ptr에 저장

&nbsp;			cout << ptr << endl; //val의 메모리 주소 출력

&nbsp;			cout << \*ptr << endl; //ptr의 역참조, val의 값을 출력

&nbsp;		}

&nbsp;		```

&nbsp;	\* ### 역참조(dereference)

&nbsp;		\* 포인터에 \*을 붙이는 를 통해 주소값의 변수에 직접 접근 가능하며, 수정도 해당 주소값 원래 변수에 적용됨.

&nbsp;		\* 포인터가 가리키는 위치에서 연산을 통해 다른 주소로 이동할 수 있음.

&nbsp;			\* 자료형마다 메모리 크기가 다르므로 포인터도 자료형을 통해 움직여야하는 크기를 알기위해 자료형을 가짐.

&nbsp;			\* 배열이 아니어도 움직일 수 있으나, 동적할당 된 메모리나 구조에 대해 파악이 되어있는 구역이 아니면 다른 변수를 가리키거나 잘못된 메모리 구역에 접근할 수 있음.

&nbsp;			```c++

&nbsp;				int main(){

&nbsp;					int val\[3] = {5,10,15};

&nbsp;					int\* ptr = \&val\[0]; 

&nbsp;					cout << \*ptr << endl; //5 출력

&nbsp;					(\*ptr)++; //val이 6으로 증가

&nbsp;					cout << \*ptr << endl; //6 출력

&nbsp;					

&nbsp;					cout << \*(ptr + 1) << endl; //10 출력

&nbsp;					cout << \*(ptr + 2) << endl; //15 출력

&nbsp;				}

&nbsp;			```

&nbsp;	\* ### 배열 포인터(Array Pointer)

&nbsp;		\* 배열을 가리키는 포인터, 배열을 탐색하는데 사용 가능.

&nbsp;		\* 배열 변수는은 배열 첫번째 원소의 주소로 변환된다는 점에서, 주소를 다루는 변수인 포인터와 유사.

&nbsp;		\* 다만 차이점도 존재:

&nbsp;			\* 자료의 사이즈: 배열은 배열 전체에 대한 사이즈를 가지고, 포인터는 자료 1개에 해당하는 사이즈를 가짐.

&nbsp;			\* 변경 가능 여부: 그리고 배열 이름에는 새로운 값을 할당하면 배열에 대한 참조를 잃어버리기에 할당할 수 없지만, 포인터 변수에는 필요에 따라 다른 주소를 할당할 수 있음.

&nbsp;			```c++

&nbsp;			int main(){

&nbsp;				int val\[5] = {1,2,3,4,5};

&nbsp;				int (\*ptr)\[5] = \&val;	//val 주소를 저장

&nbsp;				cout << \&val\[0] << endl; //val\[0] 주소를 출력

&nbsp;				cout << ptr << endl; //val 주소 => val\[0] 주소를 출력

&nbsp;				//모두 val\[2] 출력

&nbsp;				cout << \*(\*ptr + 2) << endl;//포인터 주소값 이동 (\*ptr은 val 안의 int값이 아닌 배열 val, 즉 주소를 반환)

&nbsp;				cout << (\*ptr)\[2] << endl;		//인덱스 문법 활용

&nbsp;			}

&nbsp;			```

&nbsp;	\* ### 포인터 배열(Pointer Array)

&nbsp;		\* 포인터로 이루어진 배열도 가능, 여러개의 포인터를 가짐.

&nbsp;		\* 원소인 각 포인터가 할당을 통해 각각 다른 변수를 가리키도록 설정할 수도 있음.

&nbsp;			```c++

&nbsp;			int main(){

&nbsp;				int a = 1, b = 2, c = 3, d = 4, e = 5;

&nbsp;				int val = 10;

&nbsp;				int\* ptr\[5] = {\&a,\&b,\&c,\&d,\&e};

&nbsp;				cout << \*(ptr\[0]) << endl; //a 출력

&nbsp;				cout << \*(ptr\[1]) << endl; //b 출력	

&nbsp;				ptr\[0] = \&val;

&nbsp;				cout << \*(ptr\[0]) << endl; //val 출력

&nbsp;				cout << \*(ptr\[1]) << endl; //b 출력

&nbsp;			} 

&nbsp;			```

&nbsp;	\* ### 이중 포인터(Double Pointer) 

&nbsp;		\* 포인터를 가리키는 포인터. (3중, 다중포인터도 문법상 문제는 없으나, 가독성엔 좋지 않음)

&nbsp;			```c++

&nbsp;			int main(){

&nbsp;				int val = 1;

&nbsp;				int\* ptr = \&val;	//val 주소를 저장

&nbsp;				int\*\* ptrX = \&ptr; //ptr 주소를 저장, ptr을 가리킴.

&nbsp;				

&nbsp;				cout << ptr << endl; //val의 주소값

&nbsp;				cout << \*ptrX << endl;		//val의 값 => val의 주소값

&nbsp;				

&nbsp;				cout << \*ptr << endl; //ptr의 역참조 => val

&nbsp;				cout << \*\*ptrX << endl;		//ptrX의 역참조의 역참조 => ptr의 역참조 => val

&nbsp;			}

&nbsp;			```

\* ## 레퍼런스(Reference)

&nbsp;	\* C++에서 포인터에 이어 도입된 문법으로, 주소를 다룬다는 점에서 유사하지만 더 간결하고 안전한 문법을 제공.

&nbsp;	\* 기존에 이미 존재하는 변수에 다른 이름으로 별명을 지어주는 것과 같음.

&nbsp;	\* 포인터와 비슷하다 해도 항상 유효하다는 강한 제약을 만드는 몇가지 차이점이 있다.

&nbsp;		\* 선언과 동시에 초기화되어야 한다.

&nbsp;		\* 나중에 다른 주소 값으로 수정할 수 없다.

&nbsp;		\* 유효한 null이 아닌 객체를 이용해 초기화 되야 한다. nullptr에 바인딩 될 수 없다. 

&nbsp;		\* 레퍼런스 자체에 대한 포인터 연산이 없다. \*를 통한 역참조는 안되며(이미 역참조인 형태), \&를 이용한 주소값 접근은 레퍼런스가 가리키는 일반 변수의 주소값을 반환한다. 때문에 사용할 때 일반 변수처럼 사용된다.

&nbsp;		```c++

&nbsp;		int main(){

&nbsp;			int val = 1;

&nbsp;			int\* ptr = \&val;	//val 주소를 저장

&nbsp;			//자료형\& 변수이름 = 가리킬변수

&nbsp;			int\& ref = val;	//val을 저장

&nbsp;			

&nbsp;			//아래 두 코드 모두 동일하게 작동, val의 주소값을 출력

&nbsp;			cout << ptr << endl;

&nbsp;			cout << \&ref  << endl;

&nbsp;			

&nbsp;			//아래 두 코드 모두 동일하게 작동, 1을 출력

&nbsp;			cout << \*ptr << endl;

&nbsp;			cout << ref  << endl;

&nbsp;			

&nbsp;			ref++;	//val을 증가, (\*ptr)++와 동일

&nbsp;			cout << ref << endl; //2를 출력

&nbsp;		}

&nbsp;		```

&nbsp;	\* 상수 레퍼런스

&nbsp;		\* 상수 레퍼런스로 선언하면 읽기만 가능하고 변수 수정은 불가능

&nbsp;		```c++

&nbsp;		int main(){

&nbsp;			int val = 1;

&nbsp;			const int\& ref = val;	//val을 저장

&nbsp;			cout << ref  << endl; //1을 출력

&nbsp;			

&nbsp;			ref++;	//에러!

&nbsp;		}

&nbsp;		```


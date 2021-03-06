# Solidity Function & Event
  - function
  - event

---

## function
  ### 1. 함수의 선언
  - `function` 을 이용해 선언
  - 인자명을 보통 `_` (언더바) 로 시작해서 전역변수와 구별하는 것이 관례

  ```javascript
  contract Greeter {
      function createInfo(string _name, uint _age, string _phone){

      }
  }
  ```

  ### 2. 함수의 접근 제어자
  - 해당 함수를 부를 수 있는 어카운트 범위를 제한 (어카운트는 컨트랙트 어카운트(CA), 외부소유어카운트(EOA) 가 존재)
  - 함수의 가시성 종류
    - `public` : 모든 곳에서 호출 가능하며 어카운트에 대한 제한도 없다. </br>호출에 사용된 매개변수가 항상 메모리에 기록되기 때문에 가스가 많이 소모될 수 있다.
    - `private` : 함수를 선언한 컨트랙트에서만 호출 가능
    - `external` : 외부소유어카운트에서만 호출 가능, 호출에 사용된 매개변수들은 calldata 영역에 기록 (인터페이스 사용시 많이 이용됨)
    - `internal` : 함수를 선언한 컨트랙트 및 상속받은 컨트랙트에서만 호출 가능
  - tip> 기본적으로는 `private` 으로 선언하고, 공개할 함수만 `public` 으로 선언하도록 한다.

  ```javascript
  contract Greeter {
      function createInfo(string _name, uint _age, string _phone) public {

      }
  }
  ```

  ### 3. 함수의 반환값
  - `returns` 를 선언하여 반환값 종류를 선언
  - 함수 내부에 `return` 을 활용하여 특정 값을 반환
  - 솔리디티에서는 여러가지 값을 리턴할 수도 있다. (ex> 크립토 키티의 `getKitty` )
    - 이 경우 `(a,b,c) = 여러리턴함수();` 와 같이 값을 가져올 수 있다.
    - 원하는 값만 가져오고 싶은 경우 `(,,c) = 여러리턴함수();` 와 같이 가져올 수 있다.

  > return 값이 한개인 경우

  ```javascript
  contract Greeter {
      function createInfo(string _name, uint _age, string _phone) public returns(string) {
          return "Success";
      }
  }
  ```

  > return 값이 여러개인 경우 (크립토키티의 getKitty_ 일단 public으로 바꿔놈(원래는 external))

  ```javascript
  function getKitty(uint256 _id) public view returns (
      bool isGestating,
      bool isReady,
      uint256 cooldownIndex,
      uint256 nextActionAt,
      uint256 siringWithId,
      uint256 birthTime,
      uint256 matronId,
      uint256 sireId,
      uint256 generation,
      uint256 genes
  ) {
      Kitty storage kit = kitties[_id];

      // if this variable is 0 then it's not gestating
      isGestating = (kit.siringWithId != 0);
      isReady = (kit.cooldownEndBlock <= block.number);
      cooldownIndex = uint256(kit.cooldownIndex);
      nextActionAt = uint256(kit.cooldownEndBlock);
      siringWithId = uint256(kit.siringWithId);
      birthTime = uint256(kit.birthTime);
      matronId = uint256(kit.matronId);
      sireId = uint256(kit.sireId);
      generation = uint256(kit.generation);
      genes = kit.genes;
  }

  // 값을 가져올 때
  function process() external {
    uint kittyGenes;
    uint kittyBirth;
    (,,,,,kittyBirth,,,,kittyGenes) = getKitty(주소);
  }
  ```

  ### 4. 함수 상태 제어자
  - 함수 제어자는 컨트랙트에 정의된 상태 변수를 읽거나 쓰는 연산을 함수 내에서 수행할 수 있는지를 제한하기 위한 용도로 사용
  - 이를 적절히 사용해야 gas 소모량을 줄일 수 있다. (pure 와 view 는 가스를 소모하지 않는다.)
  - 함수 제어자 종류
    - `pure` : 블록체인 네트워크에 기록된 데이터에 아예 접근하지 않으며, 파라미터로 주어지지 않은 상태 변수는 읽거나 쓸 수 없다.
    - `view` : 데이터를 보기만 하고 데이터를 수정할 수 없다.
    - `payable` : 계약 계정에 외부에서 이더를 송금 받을 수 있도록 한다.
    - `non-payable` (default) : 특별히 선언하지 않으면 지정되며 이더 송금이 불가한 payable 이다.

  ```javascript
  contract exampleContract {
      uint private year = 2018;
      uint256 balance = 0;

      // view
      function getCurrentYear() public view returns (uint) {
          return year;
      }

      // pure
      function isLeapYear(uint _year) public pure returns (bool) {
          return ((_year % 4 == 0) && (_year % 100 != 0)) || (_year % 400 == 0);
      }

      // payable
      function buySomething() external payable {
        transferThing(msg.sender); // 송금하는건 나중에..
      }
  }
  ```

---

## Event & require / assert
  ### 1. Event
  - 컨트랙트가 블록체인 상에서 앱의 사용자 단에서 무언가 액션이 발생했을 때 의사소통하는 방법
  - 특정 이벤트가 일어나는지 "귀를 기울이고" 그 이벤트가 발생하면 행동을 취함

  ```javascript
  // 이벤트를 선언
  event IntegersAdded(uint x, uint y, uint result);

  function add(uint _x, uint _y) public {
    uint result = _x + _y;
    // 이벤트를 실행하여 앱에게 add 함수가 실행되었음을 알림
    IntegersAdded(_x, _y, result);
    return result;
  }
  ```

  - 이에 대한 이벤트를 자바스크립트로 구현하게 되면 다음과 같이 지정할 수 있다.

  ```javascript
  YourContract.IntegersAdded(function(error, result) {
      // 결과와 관련된 행동을 취함
  })
  ```

  ### 2. require & assert
  - __공통점__ : 특정 조건이 참이 아닐 때 함수가 에러 메시지를 발생하고 실행을 멈춤
  - __차이__
    - `require` : 함수 실행이 실패하면 남은 가스를 사용자에게 되돌려줌
    - `assert` : 함수 실행이 실패해도 남은 가스를 되돌려주지 않음
  - 일반적으로 require을 사용하고 assert는 코드가 심각하게 잘못 실행될 때 주로 사용

  ```javascript
  function sayHiToVitalik(string _name) public returns (string) {
    // _name이 "Vitalik"인지 비교한다. 참이 아닐 경우 에러 메시지를 발생하고 함수를 벗어남
    require(keccak256(_name) == keccak256("Vitalik"));
    // 참이면 함수 실행
    return "Hi!";
  }
  ```

---

## msg property
  ### 1. msg.sender
  - 현재 함수를 호출한 사람(혹은 스마트 컨트랙트)의 주소를 가리키는 전역 변수
  - 컨트랙트는 누군가가 항상 실행하므로 `msg.sender` 는 반드시 존재한다.

  > mapping 을 이용한 예제

  ```javascript
  mapping (address => uint) favoriteNumber;

  function setMyNumber(uint _myNumber) public {
    // `msg.sender`에 대해 `_myNumber`가 저장되도록 `favoriteNumber` 매핑을 업데이트
    favoriteNumber[msg.sender] = _myNumber;
  }

  function whatIsMyNumber() public view returns (uint) {
    // sender의 주소에 저장된 값을 로드
    // sender가 `setMyNumber`을 아직 호출하지 않았다면 반환값은 `0`
    return favoriteNumber[msg.sender];
  }
  ```

  ### 2. 그 외 msg 객체
  - `msg.value` : 계약주소로 보낸 이더량
  - `msg.data` : 호출 데이터
  - `msg.gas` : gas limit 에서 함수를 호출하고 남은 가스량
  - 위에서 만든 것처럼 `msg.xxx` 의 형태로 메세지 프로퍼티를 가져올 수 있다.
  - 이는 Web 혹은 다른곳에서 컨트랙트로 transaction을 생성할때 만들어지는 것과 동일!!

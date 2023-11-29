## 시작할 때

> 0. **인텔리제이 버전 확인: 프로젝트, 빌드툴**
> 1. 깃허브d에서 **Fork**하기 
> 2. iterm에서 디렉토리 만들어 **Clone**하기 `git clone http:~~~`
> 3. **seedspirit 브랜치** 만들고 전환해서 작업 `git branch -c seedspirit `
> 4. docs 생성하고 pull request 보내지는지 확인 `git push origin seedspirit` (`git push -u origin seedspirit` 쓰면 이후 git push만 해도 됨)



### 구현 순서
기본 프로젝트 구조
```
프로젝트이름
├── Constants
│   └── DomainConstants.java
│
├── Controller
│   └── Controller.java
│
├── Message
│   ├── Exception
│   │       ├──  ExceptionPrefix.java
│   │       └──  CommonValidatorErrMsg.java
│   └── PromptMsg
│           ├──  InputViewPromptMsg.java
│           └──  OutputViewPromptMsg.java
│
├── Model
│
└── View
    ├──  InputView
    │       ├──  InputValidtor
    │       └──  InputView.java
    │
    └──  OutputView
            └──  OutputView.java
```

- InputView부터 구현해도 되지만, InputView 구현은 (validation 포함) 30분 내로 하도록 한다.
- Constants와 Message에 처음부터 너무 힘을주지 않는다. 중요한 건 핵심로직



### Validator
- 필요한 거 가져다 쓰기
- 우선 출력 메시지에서 포맷팅은 신경쓰지 않고, 이후에 시간 남을 때 포맷팅을 고려한다

```

// 빈 입력인지 체크
public void isNotEmpty(String userInput) throws IllegalArgumentException{
    if(userInput.isEmpty()){
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 공백 있는지 체크
public final String WHITE_SPACE = " ";

public void containsNoWhiteSpace(String userInput) throws IllegalArgumentException{
    if(userInput.contains(WHITE_SPACE)){
            throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
        }
    }


// 특정 포맷에 맞는지 확인
private final String MOVE_DIR_REGULAR_EXPRESSION = "^[UD]$"; 특정문자 하나만 와야 한다면 
private final String NUMBER_REGULAR_EXPRESSION = "^[0-9]+$";
private final String STRING_REGULAR_EXPRESSION = "^[a-zA-Z]+$"; // 한글까지 포함 한다면 [a-zA-Z가-힣]


public void isRightFormat(String userInput) throws IllegalArgumentException{
    if(!userInput.matches(적절한_정규표현식)){
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 입력된 숫자가 특정 범위 내에 있는지
public void isBetweenValidRange(String userInput) throws IllegalArgumentException{
    if(Integer.parseInt(userInput) < 최소값상수
            || Integer.parseInt(userInput) > 최대값상수){
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 리스트 안의 원소의 갯수가 맞는지
public void areNamesMoreThanTwo(List<String> userInputList) throws IllegalArgumentException {
    if (userInputList.size() < 최소값상수 || userInputList.size() > 최대값상수) {
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 리스트 내 문자열 원소의 길이가 유효 범위인지
public void isNameLengthInValidRange(List<String> userInputList) throws IllegalArgumentException {
    for (String userInput : userInputList) {
        if (userInput.length() < 최소값상수 || userInput.length() > 최대값상수) {
            throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
        }
    }
}


// 입력된 리스트 안에 중복 원소가 없는지
public void isNotDuplicated(List<String> userInputList) throws IllegalArgumentException {
    Set<String> userInputSet = new HashSet<String>(userInputList);
    if (userInputSet.size() != userInputList.size()) {
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}

```

```

public interface Validator {

    public void validate(String userInput);

}

```


- Validator 인터페이스로 추상화했을 경우 아래 함수들 사용 가능. 적절한 거 가져다가 쓰기 (왠만하면 2번째 추천)

```
private String getValidInput(String message, Validator validator) {
        System.out.println(message);
        String userInput = Console.readLine();
        validator.validate(userInput);
        return userInput;
    }



private String getValidInputWithRetry(String message, Validator validator) {
    while (true) {
        try {
            System.out.println(message);
            String userInput = Console.readLine();
            validator.validate(userInput);
            return userInput;
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}


private <T> T getValidInputWithRetryAndFunctionApply(String message, Validator validator, Function<String, T> converter) {
    while (true) {
        try {
            System.out.println(message);
            String userInput = Console.readLine();
            validator.validate(userInput);
            return converter.apply(userInput);
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}

```


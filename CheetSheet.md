## 시작할 때

> 0. **인텔리제이 버전 확인: 프로젝트, 빌드툴**
> 1. 깃허브에서 **Fork**하기 
> 2. iterm에서 디렉토리 만들어 **Clone**하기 `git clone http:~~~`
> 3. **seedspirit 브랜치** 만들고 전환해서 작업 `git switch -c seedspirit `
> 4. docs 생성하고 pull request 보내지는지 확인 `git push origin seedspirit` (`git push -u origin seedspirit` 쓰면 이후 git push만 해도 됨)
> 5. CheetSheet 복붙할 때 세부적인 것들(인자 이름 등) 신경쓰고 바꾸기 - 특히 자료형 네이밍이 변수명에 있는 경우



### 구현 순서
기본 프로젝트 구조
```
프로젝트이름
├── Constants
│   └── DomainConstants.java (enum 혹은 java 중 상황에 맞게 선택)
│
├── Controller
│   └── Controller.java
│
├── Message
│   ├── Exception
│   │       ├──  ExceptionPrefix.java (enum)
│   │       └──  CommonValidatorErrMsg.java (enum)
│   └── PromptMsg
│           ├──  InputViewPromptMsg.java (enum)
│           └──  OutputViewPromptMsg.java (enum)
│
├── Model
│
└── View
    ├──  InputView
    │       ├──  InputValidtor
    │       │       └──  CommonValidator.java    
    │       └──  InputView.java
    │
    └──  OutputView
            └──  OutputView.java
```

- InputView부터 구현해도 되지만, InputView 구현은 (validation 포함) 30분 내로 하도록 한다.
- Constants와 Message에 처음부터 너무 힘을주지 않는다. 중요한 건 핵심로직

### InputView

- 기능 명세를 작성한 뒤 프롬프트 메시지부터 만들면서 전체적인 데이터를 파악한다

#### InputViewPromptMsg.java & OutputViewPromptMsg.java
```
public enum InputViewPromptMsg {
   
   // 메시지 입력

    private String message;

    InputViewPromptMsg(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}


public enum OutputViewPromptMsg {
    
    // 메시지 입력
    
    private String message;
    
    OutputViewPromptMsg(String message){
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
```


### Exception

- 프리픽스 먼저 만들고 예외처리 메시지 만들기
- 프리픽스 외 enum 클래스는 commonvalidatorerrmsg 참고
- 에러 메시지 안 헷갈리게 조심하기 & 꼭 .getMessage()로 출력하기
- Validation 목록을 보고 common 이라는 네이밍을 쓸 것인지, 안 쓸 것인지 확인해야 한다

```
public enum ExceptionPrefix {
    ERROR_PREFIX("[ERROR] ");

    private final String message;

    ExceptionPrefix(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}

public enum CommonValidatorErrMsg {
    ASK_NO_EMPTY_INPUT("빈칸은 허용되지 않습니다."),
    ASK_NO_WHITE_SPACE("공백 없이 입력해주세요.");

    private String message;

    CommonValidatorErrMsg(String message) {
        this.message = ExceptionPrefix.ERROR_PREFIX.getMessage() + message;
    }

    public String getMessage() {
        return message;
    }
}

```



### Validator
- 필요한 거 가져다 쓰기
- 우선 출력 메시지에서 포맷팅은 신경쓰지 않고, 이후에 시간 남을 때 포맷팅을 고려한다

#### CommonValidator & ErrMsg

```

public enum CommonValidatorErrMsg {
    ASK_NO_EMPTY_INPUT("빈칸은 허용되지 않습니다."),
    ASK_NO_WHITE_SPACE("공백 없이 입력해주세요.");

    private String message;

    CommonValidatorErrMsg(String message) {
        this.message = ExceptionPrefix.ERROR_PREFIX.getMessage() + message;
    }

    public String getMessage() {
        return message;
    }
}


public class CommonValidator {

    private CommonValidator() {

    }

    // 기술적으로는 빈 칸이 아니지만, 실질적으로 빈 칸인 케이스를 잡기 위해 사용
    public static void isBlankInput(String userInput) throws IllegalArgumentException {
        if (!userInput.isEmpty() && userInput.isBlank()) {
            throw new IllegalArgumentException(ASK_NO_EMPTY_INPUT.getMessage());
        }
    }


    // 기술적으로 빈 입력을 잡아내기 위해 사용
    public static void isEmptyInput(String userInput) throws IllegalArgumentException {
        if (userInput.isEmpty()) {
            throw new IllegalArgumentException(ASK_NO_EMPTY_INPUT.getMessage());
        }
    }

    // 중간에 공백(띄어쓰기)가 있는지 확인
    public static void containsNoWhiteSpace(String userInput) throws IllegalArgumentException {
        if (userInput.contains(" ")) {
            throw new IllegalArgumentException(ASK_NO_WHITE_SPACE.getMessage());
        }
    }
}
```

```


// 특정 포맷에 맞는지 확인
private final String MOVE_DIR_REGULAR_EXPRESSION = "^[UD]$"; 특정문자 하나만 와야 한다면 
private final String NUMBER_REGULAR_EXPRESSION = "^[0-9]+$";
private final String STRING_REGULAR_EXPRESSION = "^[a-zA-Z]+$"; // 한글까지 포함 한다면 [a-zA-Z가-힣]

public void validate(String userInput) {
        isBlankInput(userInput);
        isEmptyInput(userInput);
        containsNoWhiteSpace(userInput);

    }


private void isRightFormat(String userInput) throws IllegalArgumentException{
    if(!userInput.matches(적절한_정규표현식)){
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 입력된 숫자가 특정 범위 내에 있는지
private void isBetweenValidRange(String userInput) throws IllegalArgumentException{
    if(Integer.parseInt(userInput) < 최소값상수
            || Integer.parseInt(userInput) > 최대값상수){
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 리스트 안의 원소의 갯수가 맞는지
private void areNamesMoreThanTwo(List<String> userInputList) throws IllegalArgumentException {
    if (userInputList.size() < 최소값상수 || userInputList.size() > 최대값상수) {
        throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
    }
}


// 리스트 내 문자열 원소의 길이가 유효 범위인지
private void isNameLengthInValidRange(List<String> userInputList) throws IllegalArgumentException {
    for (String userInput : userInputList) {
        if (userInput.length() < 최소값상수 || userInput.length() > 최대값상수) {
            throw new IllegalArgumentException(에러_메시지_Enum.getMessage());
        }
    }
}


// 입력된 리스트 안에 중복 원소가 없는지
private void isNotDuplicated(List<String> userInputList) throws IllegalArgumentException {
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

### InputView

```
public class InputView {

    // 필요한 객체 만들기

    public InputView(){
        System.out.println(//초기화 때 내보내야 하는 기본 안내);
    }

    public Integer visitDateInput(){ // 자료형, 메서드 이름은 알아서 수정 & 알아서 반환해서 리턴
        String userInput = inputWithValidation(
                // 적절한enum.getMessage(),
                // 적절한 validator
        );

        return ;
    }
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

만약 모델에서 검증하고 다시 입력을 받아야 하는 경우
```
// 전역 변수를 안 쓰는 버전
private List<Integer> getUserInputWithValidation(UserInput userInput, CardGame cardGame){
        return receiveInputUntilValid(userInput, cardGame);
    }

private List<Integer> receiveInputUntilValid(UserInput userInput, CardGame cardGame) {
    while (true) {
        List<Integer> input = userInput.getFirstUserInput();
        if (isInputValid(input, cardGame)) {
            return input;
        }
    }
}

private boolean isInputValid(List<Integer> input, CardGame cardGame) {
    try {
        cardGame.validateIfCoordAlreadySolved(input);
        return true;
    } catch (IllegalArgumentException e) {
        System.out.println(e.getMessage());
        return false;
    }
}
```
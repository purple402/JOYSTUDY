# 객체 지향 프로그래밍의 설계 원칙
## SOLID - 1. SRP

## 단일 책임의 법칙 SRP(Single Responsibility Principle)
단일 책임의 법칙은 아래의 내용을 이야기하는 것이다.

- 클래스는 단 한 개의 책임을 가져야 한다.
- 클래스를 변경하는 이유는 단 하나여야 한다(단 한개의 책임만을 가지기 때문).
- 한 책임의 변경에 의해 다른 책임과 관련된 코드에 영향을 미치지 않도록 한다(유지보수 용이).

react 를 사용하면서 class 를 만드는 것이 아니기 때문에 완벽하게 SRP를 지키는 class를 만들지는 못했다. 하지만 기존의 코드에서 어떻게하면 SRP를 지키도록 만들 수 있을지 고민해보았다.

### AS-IS

```jsx
const join = async () => {
  if (userData.userName.length > 50) {
    alert('이름은 최대 50자 입니다.');
    return false;
  }
  if (!userData.email) {
    alert('이메일을 입력해주세요.');
    return false;
  }
  if (
    !userData.email.match(
      /^([a-zA-Z0-9_\-\.]+)@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.)|(([a-zA-Z0-9\-]+\.)+))([a-zA-Z]{2,4}|[0-9]{1,3})(\]?)$/
    )
  ) {
    alert('이메일 형식을 확인해 주세요.');
    return;
  }
  if (!userData.privateCheck) {
    alert('개인정보 취급방침에 동의해주세요.');
    return false;
  }
  if (!userData.useCheck) {
    alert('이용약관에 동의해주세요.');
    return false;
  }

  const params = {
    loginId: props.getUserData.loginId,
    password: props.getUserData.password,
    user_name: authData.user_name,
    gender: authData.sex_code === '1' ? 'M' : 'F',
    phone_number: authData.phone_number,
    birthday: authData.birth_day,
    email: userData.email,
    name: userData.userName,
  };

  const res = await loginApi.join(params);

  if (res.result) {
    alert('회원가입이 완료되었습니다.');
    navigate('/web/login');
  } else {
    alert('회원가입이 정상적으로 이뤄지지 않았습니다.');
  }
};
```

회원가입하기를 실행하는 함수에서 입력받은 데이터의 유효성 검사와 API 호출, 응답에 대한 처리까지 이루어지고있다. 유지보수시 흐름을 따라가기 어렵고, 다른 유효성 검사를 추가하려면 또다시 join 함수 내부가 길어지게 된다.

### TO-BE

```jsx
// 회원가입 정보 유효성 검사
const validateBeforeJoin = () => {
  if (userData.userName.length > 50) {
    alert('이름은 최대 50자 입니다.');
    return false;
  }
  if (!userData.email) {
    alert('이메일을 입력해주세요.');
    return false;
  }
  if (
    !userData.email.match(
      /^([a-zA-Z0-9_\-\.]+)@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.)|(([a-zA-Z0-9\-]+\.)+))([a-zA-Z]{2,4}|[0-9]{1,3})(\]?)$/
    )
  ) {
    alert('이메일 형식을 확인해 주세요.');
    return;
  }
  if (!userData.privateCheck) {
    alert('개인정보 취급방침에 동의해주세요.');
    return false;
  }
  if (!userData.useCheck) {
    alert('이용약관에 동의해주세요.');
    return false;
  }

  return true;
};

// 회원가입 시도 API 호출
const requestJoin = async () => {
  const params = {
    loginId: props.getUserData.loginId,
    password: props.getUserData.password,
    user_name: authData.user_name,
    gender: authData.sex_code === '1' ? 'M' : 'F',
    phone_number: authData.phone_number,
    birthday: authData.birth_day,
    email: userData.email,
    name: userData.userName,
    };

  return await loginApi.join(params);
}

// 회원가입
const join = async () => {
  // 유효성 검사
  const isReady = validateBeforeJoin();
  if (!isReady) return;
  
  // 회원가입 요청
  const { result: joinResult } = await requestJoin();
  if (joinResult) {
    alert('회원가입이 완료되었습니다.');
    navigate('/web/login');
  } else {
    alert('회원가입이 정상적으로 이뤄지지 않았습니다.');
  }
};
```

join 내에서 이루어지던 모든 것들을 기능에 따라 각각의 함수로 분리했다.

유효성 검사는 validateBeforeJoin, 회원가입 요청은 requestJoin 으로 분리했더니 join 함수 안에서 어떤 일들이 이뤄지는지 더 확인하기 편하고, 특정 기능을 유지보수 할 때 더 쉽게 수정할 곳을 찾을 수 있다.
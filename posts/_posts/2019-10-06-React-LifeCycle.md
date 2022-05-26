---
layout: post
title: "리액트 라이프사이클"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "React"
comments: true
---

## 1. 생명주기
리액트에서 아용되는 모든 컴포넌트들은 라이프사이클을 따라서 통해 생성, 업데이트, 제거 된다.
리액트 개발시 생명주기에 대해 이해하는것은 중요하다고 생각하여 이번기회에 정리해보고자 한다.
그리고 개발하면서 새로 알은 지식이 있디면 추가적으로 수정해서 보완해 나가려 한다.<br/>
함수형 컴포넌트에서도 hook의 개념이 도입되면서 생명주기를 이용한 코딩이 가능해졌지만
이 포스팅에선 클래스 기반 컴포넌트의 생명주기를 다루어보려 한다

### 2. render 단계
#### 1. constructor()
```javascript       
constructor(props) {
    super(props)
    this.state = {
        isLoading: true,
        flower: {},
        newCommentState: false,
        commentsCount: 6,
        lastCommentPage: 1,
        lastCommentPosition: 0
    }
    this.readFlowerData = readFlowerData.bind(this)
    this.newComment = newComment.bind(this)
    this.deleteComment = deleteComment.bind(this)
    this.openNewComment = openNewComment.bind(this)
    this.closeNewComment = closeNewComment.bind(this)
    this.readComments = readComments.bind(this)
    this.checkComments = checkComments.bind(this)
    this.errorHandler = errorHandler.bind(this)
}
```
리액트에서 아용되는 모든 클래스 기반 컴포넌트들은 생성자를 거친후 rendering 된다.<br/>
리액트는 Reconciliation 알고리즘을 통해 constructor 호출 횟수를 최소화한다.


#### 2. getDerivedStateFromProps(nextprops, prevState)
```javascript       
getDerivedStateFromProps(nextprops, prevState) {
    if (nextProps.location !== this.props.location) 
        return this.reload(nextProps.location.search)
}
```
getDerivedStateFromProps은 이전버전의 리액트에서 사용되었던 componentWillReceiveProps의 대체제이다.<br>
새로운 props를 받았을때 그에따른 state을 설정하기 위한 목적으로 사용되며 setState() 함수가 아닌 변경할 state 객체를 
반환해야 한다. props가 변경되어 update 이벤트 발생시, constructor 실행시 다음으로 실행된다.

#### 3. shouldComponentUpdate(nextProps, nextState)
update 이벤트 발생시 다시 rerendering 할지 결정한다.
default로 true를 반환하며 사용자가 로직을 짜서 경우에따라 false를 반환하도록하여 성능을 향상시킬수 있다.

#### 4. render()
```javascript  
render() {
    return (
        <div className="search" onClick={() => document.getElementsByClassName('menu')[0].style.display = 'none'}>
            <SearchHeader />
            {getQuery.parse(this.props.location.search).query !== undefined ?
                (<SearchSubheader />) : (<BlankHeader />)
            }
            <div className="result">
                {this.state.isLoading ? (
                    <div className="loading">
                        <p>loading...</p>
                    </div>
                ) : (
                this.state.flowerData.map((value, index) => {
                    return <Link 
                            to={"/flowers/" + value.id} 
                            key={index}><SearchFlowerDetail flower={value} 
                            /></Link>
                        })
                    )}
            </div>
        </div>
    );
}
``` 
render 함수를 통해 컴포넌트를 DOM에 탑재한다. 이때부터는 DOM을 통해 컴포넌트에 접근할수 있다.
### 3. commit 단계
#### 5. componentDidmount()
```javascript  
componentDidMount() {
    this.reload(this.props.location.search)
    document.getElementsByClassName('result')[0].addEventListener('scroll', (event) => this.additonalLoading(event))
}
```
mount과정을 진행하면서 render가 끝난후 refs 와 DOM updating 작업이 끝나면 componentDidMount()가 실행된다.

#### 6. componentDidUpdate()
```javascript  
componentDidUpdate() {
    this.reload(this.props.location.search)
    document.getElementsByClassName('result')[0].addEventListener('scroll', (event) => this.additonalLoading(event))
}
```
update과정을 진행하면서 render가 끝난후 refs 와 DOM updating 작업이 끝나면 componentDidUpdate()가 실행된다.

### 4. 참고자료
<a href="https://velog.io/@kyusung/%EB%A6%AC%EC%95%A1%ED%8A%B8-%EA%B5%90%EA%B3%BC%EC%84%9C-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%99%80-%EB%9D%BC%EC%9D%B4%ED%94%84%EC%82%AC%EC%9D%B4%ED%81%B4-%EC%9D%B4%EB%B2%A4%ED%8A%B8">https://velog.io/@kyusung/%EB%A6%AC%EC%95%A1%ED%8A%B8-%EA%B5%90%EA%B3%BC%EC%84%9C-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%99%80-%EB%9D%BC%EC%9D%B4%ED%94%84%EC%82%AC%EC%9D%B4%ED%81%B4-%EC%9D%B4%EB%B2%A4%ED%8A%B8</a>

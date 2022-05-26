---
layout: post
title: "쉘스크립트 I"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
comments: true
---

```bash
#!/bin/bash

numbers=($@ $@)

for ((i=0; i < ${#numbers[@]}; i++))
do
  for ((j=0; j < ${#numbers[@]}-i-1; j++))
  do
    if [ ${numbers[j]} -gt ${numbers[$((j+1))]} ]
    then
      temp=${numbers[j]}
      numbers[$j]=${numbers[$((j+1))]}
      numbers[$((j+1))]=$temp
    fi
  done
done


printf '%s\n' "${numbers[@]}"
```
위의 코드는 버블소트를 bash 스크립트로 구현한것이다. 스크립트를 한줄한줄 읽어가며 bash의 규칙이나 기능을 알아보고자한다.

```bash
#!/bin/bash
```
만약 첫줄이 #!/bin/bash 로 시작하면 현재실행되고 있는 shell에서 스크립트를 실행하기 위한 프로세스가 fork된후 스크립트 실행을 위한 인터프리터로 bash가 사용된다. 쉘과 함께 인자를 넘기는것도 가능하다.

```bash
numbers=($@)
```

대입연산은 무조껀 붙여서 한다. 스크립트를 실행할때 공백기준으로 나누어 해석을 하는데 이때 앞의 operend가 명령어로 인식되기 때문이다. 아무 지시어 없이 () 가 사용되면 배열을 뜻한다. () 안에 있는 $@를 통해 전달된 argument들로 치환된다.

```bash
for ((i=0; i < ${#numbers[@]}; i++))
do
...
done
```

(( ... )) 구문은 특수구문으로 연산자 escape나 변수를 quote해서 사용하는 제약이 사라진다. 변수이름에도 $ 문자를 붙이지 않아도 되지만 매개변수 확장과 명령치환시에는 $가 필요하다. 위와 같이 for loop 도 c style로 작성할수 있다.
#numbers[@] 를 통해 배열의 모든 원소의수를 연산할수 있다. 

```bash
if [ ${numbers[j]} -gt ${numbers[$((j+1))]} ]
then
  temp=${numbers[j]}
  numbers[$j]=${numbers[$((j+1))]}
  numbers[$((j+1))]=$temp
fi
```

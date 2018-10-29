디버깅
===============

* 항상 네트워크를 최신화하세요. 이전 작업을 삭제하려면 `deleteNetwork.sh` 스크립트를 사용하세요.

* 아래와 같은 오류가 나타난다면, 환경이 제대로 설정되지 않은 것이므로 kubectl 명령어가 작동하지 않을겁니다.
  ```
  $ kubectl get pods
  The connection to the server localhost:8080 was refused - did you specify the right host or port?
  ```

* 아래와 같은 오류가 나타난다면, 이 peer가 채널에 접속하지 않은 것이므로 당신의 쿼리가 작동하지 않을 것입니다.

  ![](images/error1.png)

* 다음과 비슷한 상황이 일어난다면,

  ![](images/error2.png)

  명령어에 오류가 있는 것입니다. 이 스크린샷에는 전달인자 이전에 '-c'가 빠져있습니다.

* 아래와 같은 오류가 나타난다면,

  ![](images/error3.png)

  설치에 문제가 있는 것입니다. 최신버전으로 설치하세요.

* 디버깅하기 위해서 컨테이너의 로그를 확인하고자 한다면 다음의 명령어를 입력하세요.

  ```
   $ kubectl get pods --show-all                  # Get the name of the pod
   $ kubectl logs [pod name]                      # if pod has one container
   $ kubectl logs [pod name] [container name]     # if pod has more than one container
  ```

* chaincode 인스턴스화 중에 이런 상황을 보게 되면,

  ![](images/error4.png)

  이미 peer가 인스턴스화한 것입니다. 계속 시도한다면 위와 같은 에러가 나타날 것입니다. 위 에러를 무시하고 계속 전송할 수 있습니다. (invoke/query).  

* 아래와 같은 에러가 나타난다면,

  ![](images/error5.png)

 네트워크 때문에 일어나는 간헐적인 문제입니다. 네트워크를 삭제하고 조금 후에 새로고침하세요.  

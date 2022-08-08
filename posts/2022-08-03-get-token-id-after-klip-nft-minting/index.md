# Klaytn Transaction Hash를 통해 NFT Token ID 조회하기


* 목표
Klip에서 제공하는 Mint Card To User API를 호출하면 결과값으로 클레이튼 네트워크의 Transaction Hash를 반환하다. 반환된 Transaction Hash는 [[https://scope.klaytn.com/][Klaytn Scope]]에 입력해서 어떤 작업을 수행했는지 내용을 상세하게 확인할 수 있는데 이 때 발생된 NFT의 Token ID도 확인할 수 있다.

Klaytn Scope에서 확인했던것처럼 [[https://ko.docs.klaytn.foundation/dapp/sdk/caver-java][Caver]]를 이용해서 반환된 *Transaction Hash를 토대로 민팅된 NFT의 Token ID를 조회하는 법을 알아보자.* 이 문서에서는 Kotlin을 기준으로 설명한다.

* 작업
** Mint Card to User
Klip Docs를 토대로 [[https://docs.klipwallet.com/rest-api/rest-api-card-minting#send-card-to-user][Mint Card to User API]]를 호출한다. 이 API는 참고로 Ground X의 승인을 받아야 사용이 가능하다. API 호출 방법은 이 문서의 범위를 벗어나므로 설명하지 않겠다. 호출에 성공하면 아래와 같은 값을 얻을 수 있다.
#+BEGIN_SRC json
{
  "hash": "0xXXXXXXXXXXXXXXX"
}
#+END_SRC
** Get Token ID
위에서 얻는 값을 토대로 Token Id를 조회해보자. 아래의 코드는 [[https://forum.klaytn.foundation/t/klay-gettransactionreceipt-klay-getlogs-tokenid/1957/2][이 곳]]의 내용을 토대로 작성했다. 로직에 대한 자세한 설명은 링크를 참고하자.

#+BEGIN_SRC kotlin
private fun getNftTokenId(mintTransactionHash: String): Long? {
    val caver = Caver("https://public-node-api.klaytnapi.com/v1/cypress") // 테스트넷: https://public-node-api.klaytnapi.com/v1/baobab
    val result = caver.rpc.klay
        .getTransactionReceipt(mintTransactionHash)
        .send()
        .result ?: return null

    val hexadecimalTokenId = result
        .logs
        .first()
        .topics[3]

    return hexadecimalTokenId.substring(2).toLong(16) // 16진수를 10진수로 변환.
}
#+END_SRC

만약 민팅된 Token ID가 =15= 일 경우 =hexadecimalTokenId= 변수는 =0x00000000000000000000000f= 값을 가진다. 이는 16진수 표기법으로, 앞쪽의 =0x= 텍스트를 제거(=substring(2)=)한 후 ~toLong(16)~ 또는 =toInt(16)= 메서드를 이용해서 16진수를 10진수로 변환할 수 있다.

참고로 *민팅 직후 =send()= 메서드를 호출하면 =result= 값으로 null이 반환된다.* 그러니 null이 반환되면 일정 간격으로 메서드를 재호출하는 로직을 추가하길 권장한다.

* Other
위의 과정이 귀찮다면 [[https://www.klaytnapi.com/ko/landing/main][Klaytn API Service(KAS)]]를 이용해서 Token ID를 얻는 방법이 있지만 무료 플랜일 경우 하루 호출량에 제한이 있다.


---
emoji: ''
title: 'Blockchain: 용어 정리'
date: '2022-02-17 22:00:00'
author: seoyul
tags: blockchain
categories: BlockChain
---

### airdrop 
광고하기 위해서 무료로 자신들의 토큰을 나눠줌 
(e.g uniswap은 이전에 protocol을 사용한 사용자들의 wallet address를 list up해서(testers 혹은 early adaptor에게) 자신들의 token을 무료로 전송했다.)

### proof of work & proof of stake
proof of work란 특정한 수를 맞출 때까지 복잡한 퍼즐을 추측하고 검사하는 과정을 말한다.
비트코인에서는 랜덤으로 섞인 수와 문자를 SHA-256으로 해시하여 특정 개수는 0을 찾는 과정을 진해하고 알맞을 결과를 맞추게 되면 그 block을 solve(문제를 풀면 reward를 받게 된다.)했다고 하며, 그 block을 블록체인에 추가된다.
proof of work: race를 위해 사람들이 줄지어있다고 가정했을 때 그 사람들 중 유독 그 race에 강점을 갖고 있는 사람이 있고 경합을 할 경우 그 사람이 우승해서 reward를 받게 되며 나머지 우승하지 못한 사람들은 reward를 받지 못한 채 에너지를 낭비하게 된다.

but, proof of stake: race를 위한 사람들중 단 한명만을 뽑아서 그 race를 진행하게 하여 완료 후 reward를 받게 되고 reward를 받지 못한 사람들은 race를 진행하지 않았기 때문에 에너지를 낭비하지 않게 된다.

### staking 
proof of work에 참여하기 위해서는 coin을 담보로 lock을 걸어서 사용하지 못하게 하고 network가 mining을 할 수 있도록 선택할 때까지 기다린다. 후에 선택되어 성공적으로 mining을 하게 된다면 Staking reward를 받게 된다. 만약 틀리게 되면 locked 된 코인을 다른 참여자에게 나눠주게 된다.

코인을 담보처럼 locking up을 하는 것을 staking이라고 한다.

### liquidity pool
한 토큰을 다른 토큰으로 교환하고 싶을 경우 예전 방식을 이용하게 되면 갖고 있는 토큰을 원하는 가격에 교환하기 위해 해당 가격에 교환하길 원하는 사람이 나타나기까지 기다려야한다.

liquidity pool은 50:50의 ratio로 두개의 asset을 담고있는 pool이고 Auto Market Maker 알고리즘 공식인 x*y=k를 사용한다.
Investor가 해당 pool에 토큰을 제공할 수 있으며 investor는 교환하기를 원하는 사람이 토큰을 교환할 때 지불하는 fee를 얻을 수 있다.(pool에 재공되어있는 비율에 따라 investor들이 fee를 나눠갖게 된다.)

### AMM(Auto Market Maker)
50:50의 liquidity pool ratio를 이용하기 위해 x*y=k 의 공식(x와 y 는 수량을 의미하며 k는 constant로 항상 똑같이 유지되며 바뀌지 않는다)을 이용하며 누군가 한 토큰을 제공하고 다른 토큰이 빠지게 되면 제공해서 많아진 토큰은 가격이 내려가고 수가 줄어든 토큰은 가격이 올라가게 된다.

### arbitrage trading
한 pool 에서 trader 대량으로 ether를 제공하여 ether 가격이 낮아져서 250달러인데  다른 pool에서 ether를 300달러에 팔 수 있다면 해당 pool에서 ether를 구매해서 다른 pool에 차액을 얻고 파는 방식을 말한다.

### mining pool
일반적인 컴퓨터로 ethereum을 mining하기 위해서는 상당히 오랜 시간이 걸리는데, mining을 진행하여 당장 reward를 받기 위해서 mining을 하고자하는 많은 사람들울 모아서 그 block에서 그룹지어 한번에 mining을 한다. solution을 찾게되어 reward를 받으면 그것을 나누어 갖게된다.

짧은 시간 내에 reward를 받는 것이 목적이다.

### rug pull
개발자가 발행한 토큰의 가격을 올려놓고 투자자의 자금을 갖고 사라지는 crypto scam

### impermanent loss

### wrapped token
token을 wrap해서 다른 블록체인에서 사용할 수 있게 한다. 각각의 블록체인들은 token을 갖고 있는데 해당 블록체인에 있는 token은 다른 블록체인에서 이용하기 어렵기떄문에 만들어졌다.
Wrapped token은 100% token이 represent하고 있는 real asset이 backed된 토큰이다. (collateral 혹은 stable coin과 비슷한 의미를 지닌다.)

### gas
가스란 이더리움이 하는 일을 수치화한 단위 값이고 가스 가격에 가스 한도를 곱하면 거래 비용을 알 수 있다.(채굴자에게 돈을 지불하기 위해 Gwei라는 이더리움의 작은 값으로 단위를 나타냅니다. 이것이 가스 가격(Gas Price)이다.) 
ETH의 가격과 트랜잭션 가격을 분리함으로써 트랜잭션 비용이 비싸지는 것을 방지할 수 있고 마이너들은 네트워크 보안에 대해 보상받으므로 이더리움이 끊임없이 돌아갑니다.
트랜잭션 비용을 직접 입력하거나 사용하는 앱이 주어지는 값을 사용할 수도 있으며, 작업의 양을 알려줌으로써 채굴자들이 우선순위를 선택할 수 있게 도와준다.

### GWEI
1원이 원화의 가장 작은 단위인 것처럼 이더의 가장 작은 단위는 wei이다. 1 wei가 1이더가 되기 위해선 엄청나게 많은 wei가 필요하다.(정확하게는 10^18wei가 필요하다.)

0^9 wei는 Gwei이며 Gwei는 가스 (네트워크 거래 수수료)에 대해 말할 때 가장 일반적으로 사용된다. 가스 비용이 0.000000001 이더라고 말하기보다는 1gwei라고 간편하게 말할 수 있다.

wei에서 시작해서 10^n 으로 올리거나 이더에서 소수점으로 내려서 액면가를 표현한다.

1 Ether = 1,000,000,000,000,000,000 WEI = 1 (EXA)WEI
1 (MILLI)ETHER = 0.001 ETHER1,000,000,000,000,000 WEI = 1 (PETA)WEI
1 (MICRO)ETHER = 0.000001 ETHER1,000,000,000,000 WEI = 1 (TERA)WEI
1 (Nano)ETHER = 0.000000001 ETHER1,000,000,000 WEI = 1 (GIGA)WEI
1 (PICO)ETHER = 0.000000000001 ETHER1,000,000 WEI = 1 (MEGA)WEI
1 (FEMTO)ETHER = 0.000000000000001 ETHER1,000 WEI = 1 (KILO)WEI
1 (ATTO)ETHER = 0.000000000000000001 ETHER1 WEI

### Ref
- [Rug pull](https://cointelegraph.com/explained/crypto-rug-pulls-what-is-a-rug-pull-in-crypto-and-6-ways-to-spot-it)
- [Youtube Channel](https://www.youtube.com/channel/UCsYYksPHiGqXHPoHI-fm5sg)
- [weth](https://weth.io/kr/)

```toc

```

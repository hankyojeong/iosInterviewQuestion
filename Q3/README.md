# GCD (Grand Central Dispatch)
1. Dispatch Queue 생성
2. System Dispatch Queue 사용
3. Dispatch work item 사용
4. Dispatch group의 사용
<!-- ![AppState](./images/AppState.png) -->

## Dispatch Framework
MultiCore HW 환경에서 코드를 동시적으로 실행시키기 위한 Franework 실행시킬 작업을 시스템이 관리하는 DispatchQueue에 전달하는 것만으로 동시성 프로그래밍이 가능함

기존의 MultiThreading 프로그래밍의 경우 개발자가 직접 Thread를 생성, 관리 등의 책임을 지었지만 GCD를 사용할 경우 이를 System에게 떠넘김

## DispatchQueue
작업들을 Application의 MainThread 또는 Background Thread에서 순차적(serially) 또는 동시적(concurrently)으로 실행시키는 것을 관리하는 객체

Application은 block object(closure)의 형태로 Queue에 작업을 전달, Dispatch Queue는 작업들을 순차적(Serially) 또는 동시에(Concurrently) 실행

Dispatch Queue에 전달된 작업은 System에 의해 관리되는 Thread Pool에서 실행
Application의 Main Thread Dispatch Queue를 제외하곤 어떤 Thread가 작업을 실행할지 알 수 없음
![AppState](./images/DispatchQueueStructure.png)

Dispatch Queue에 작업 객체(work item)을 전달할 때 동기적(Sync) 또는 비동기적(Async)으로 작업을 예약

## DispatchQueue의 Serial/Concurrent, Sync/Async 란
- Serial: 요청된 **하나 이상의 작업들**을 순차적으로 실행하기 위해 선행작업이 종료되기를 기다림
- Concurrent: 요청된 **하나 이상의 작업들**을 실행할때 선행 작업의 종료를 기다리지 않고 다음 작업으로 감
- Synchronous: 작업을 요청하고 그에 대한 응답이 올 때 까지 다음 요청을 대기
- Asynchronous: 작업을 요청하고 그에 대한 응답이 오는 것을 기다리지 않고 다음 작업을 요청

Serial/Concurrent: 요청된 **여러 개의 작업**들을 순차적 또는 동시에 즉 다수의 대상에 대한 순서
Synchronous/Asynchronous: **단일 작업**에 대한 요청과 응답의 발생 시점

![AppState](./images/SerialConcurrent.png)
Concurrent의 경우 Queue의 FIFO 특성에 의해 작업들의 실행은 순서대로 이뤄지지만 선행작업이 끝나기를 기다리는 것은 아님

![AppState](./images/SyncAsync.png)


MainQueue에 전달된 작업들은 동기적으로 실행되면 안됨, UI관련 변화는 Main Queue에 전달, 그 밖에 시간이 오래 걸리는 작업들은 GlobalQueue 혹은 ConcurrentQueue를 생성 및 실행해야함

## Dispatch Queue 사용방법
생성자를 통한 직접 생성 방법과 System에서 생성해 둔 Queue를 사용하는 방법이 존재
### Custom Dispatch Queue
작업 block을 전달할 수 있는 새로운 dispatch queue를 생성

이렇게 생성한 Queue에 전달되는 작업들은 Main Thread를 제외한 나머지 Background Thread에서 실행

Parameter
- label: Queue를 고유하게 식별하기 위한 문자열 Reverse-DNS(com.example.myqueue) 형태의 이름 사용을 추천
- qos: Quality of Service, 시스템이 실행시킬 작업을 예약시 우선순위를 결정
- attributes: Queue에 관한 속성 값(ConcurrentQueue = .concurrent) 아무런 값도 전달하지 않을 시 SerialQueue로 생성, ('.initiallyInactive' = 비활성 상태의 dispatch queue를 생성, active() 함수를 호출하여 작업을 실행함)

### System Dispatch Queue
System이 생성한 Queue
- Serial Queue: main queue, 현재 Process의 Main Thread와 관련된 dispatch queue
- ConcurrentQueue: global queue, global(qos:)는 dispatch queue의 타입 함수, 지정된 qos(quality of service)로 작업을 실행시키는 global system queue를 반환, global queue는 main thread를 제외한 다른 background thread에서 실행

## DispatchQoS(Quality of Service)
작업 실행의 우선순위를 지정하는 서비스 품질에 대한 분류

**User Interactive > User Initiated > Default > Utility > Background > Unspecified**

사용자와의 상호작용 작업등은 높은 Thread 우선순위를 부여, 작업이 빠르게 실행되도록 함 background는 더 낮은 우선순위를 부여함

### User Interactive
Animation, Event 처리, UI 업데이트 등 사용자와 상호작용하는 작업들을 위한 분류, 가장 높은 우선순위를 가짐
### User initiated
사용자에게 작업의 결과를 즉각적으로 제공해야 하거나, 잠시 동안 사용자가 App을 적극적으로 사용하지 못하게 막는 작업들을 위한 분류
### Default
QoS의 기본값
### Utility
사용자가 적극적으로 확인하거나 추적하지 않는 작업을 위한 분류, 사용자가 신경쓰지 않는 오래 걸리는 작업등에 할당
### Background
계속 유지해야 하거나 정리(clean up)해야 하는 작업들을 위한 분류(ex] 동기화, 백업 등)
### Unspecified
QoS를 지정하지 않음 System이 QoS를 자동으로 추론

## Dispatch Work Item
Dispatch Queue나 Dispatch group에서 실행하기 위해 실행하려는 작업을 감싸고 있는 주체, block object 대신 work Item을 통해 작업 전달이 가능

## Dispatch Group
여러개의 작업들을(Group of Tasks) 하나의 유닛으로 관찰할 수 있게 해주는 객체. 일련의 작업들을 하나로 묶고 Group내에서 동작을 동기화(Synchronize) 시킴. 여러개의 workItem을 Group에 붙이고, 같거나 다른 Queue에서 비동기적으로 실행시킴
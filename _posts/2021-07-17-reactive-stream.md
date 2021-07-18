---
title: "Spring Webflux - (1) Reactive Stream"
categories:
  - spring
tags:
  - reactive programming
  - reactive stream
toc: true
toc_icon: "cog"
toc_sticky: true
---

#### [Reactive Stream](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md)
- Reactive Programming의 명세. 즉, non-blocking backpressure로 비동기 스트림 처리를 위한 표준
- Reactive Streams에 정의된 인터페이스 구현을 통해 Reactive Programming을 구현할 수 있음

##### Interface
- [Processor](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#4processor-code)
	```java
	public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
	}
	```
	- Publisheh와 Subscriber의 사이에서 위치하며 몇 가지 처리 단계를 유연하게 추가할 수 있음
	
- [Publisher](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#1-publisher-code)
	- Subscriber에게 데이터를 전달하는 역할
	- Publisher가 Subscriber에게 보내는 onNext의 총 수는 항상 해당 Subscriber의 Subscription이 요청한 데이터의 수보다 작거나 같아야 함
		- Publisher는 Subscriber가 요청한 수보다 많은 수의 데이터를 보낼 수 없음
		- Pusblisher가 Subscirber에 비해 빠른 경우 Subscriber에게 전달되는 데이터가 누적되는 상황을 방지하기 위한 Back-Pressure 장치
	- Publisher는 요청된 수보다 적은 수의 onNext를 호출하고 onComplete 또는 onError를 호출하여 Subscription을 종료할 수 있음
		- Publisher는 Subscriber이 요청한 수만큼의 데이터 전달을 보장하지 않음 
	- Subscriber의 onSubscribe, onNext, onError, onComplete는 직렬로 호출됨
	- Publisher.subscribe에서는 주어진 Subscriber의 onSubscribe 함수를 호출해야 하며, Subscriber가 null이 아닌 이상 정상적으로 리턴해야 함
		- Subscriber에게 어떤 시그널 보다도 onSubscriber가 먼저 호출된다는 것을 보장하며 이를 통해 Subscriber는 해당 로직에서 초기화 작업을 수행할 수 있음
	```java
	public interface Publisher<T> {
		void subscribe(Subscriber<? super T> var1);
	}
	```

- [Subscriber](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#2-subscriber-code)
	- 데이터를 구독하여 Publisher로부터 전달받는 역할
	- onNext가 호출되기 위해 subscription.request를 호출해야 함
		- 언제 얼마나 많은 데이터를 수신할 지를 결정하는 것은 Subscriber의 책임임
	- Subscriber.onComplete 와 Subscriber.onError 안에서 subscription 또는 Publisher의 어떤 함수도 호출하지 못함
	- Subscriber.onsubscribe가 호출되었을 때 이미 Active Subscription이 있는 경우 Subscription.cancel을 호출해야 함
		- 하나의 Subscriber는 둘 이상의 Publisher를 가질 수 없음
	- Subscriber는 Subscription의 모든 함수가 순차적으로 수행될 수 있게 해야함
	- Subscription.cancel을 호출한 이후에도 수신받지 못한 데이터가 있다면 하나 이상의 onNext가 호출될 수 있음 
		- cancel이 호출된 이후에 실제로 취소 처리가 되기 전까지 지연이 발생할 수 있음
	- Subscriber.onSubscribe는 최대 한 번만 호출됨
	```java
	public interface Subscriber<T> {
		void onSubscribe(Subscription var1);
		void onNext(T var1);
		void onError(Throwable var1);
		void onComplete();
	}
	```

- [Subscription](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#3-subscription-code)
	- Publisher와 Subscriber 사이에 데이터 교환을 중재하는 역할
	- Subscription.request와 Subscription.cancel은 Subscriber 안에서만 호출됨
	```java
	public interface Subscription {
		void request(long var1);
		void cancel();
	}
	```

##### 예시
- Publisher
	```java
	public class DemoPublisher implements Publisher {

		Iterable<Integer> itr = Arrays.asList(1, 2, 3, 4, 5);

		public void subscribe(Subscriber subscriber) {
			System.out.println("[Publisher:subscribe]");
			subscriber.onSubscribe(new DemoSubscription(subscriber, itr));
		}
	}
	```

- Subscriber
	```java
	public class DemoSubscriber implements Subscriber {

		private Subscription subscription;

		public void onSubscribe(Subscription subscription) {
			this.subscription = subscription;
			subscription.request(1);
		}

		public void onNext(Object o) {
			System.out.println("[Subscriber:onNext] Got : " + o);
			subscription.request(2);
		}

		public void onError(Throwable throwable) {
			System.out.println("[Subscriber:onError]");
			throwable.printStackTrace();
		}

		public void onComplete() {
			System.out.println("[Subscriber:onComplete]");
		}
	}	
	```

- Subscription
	```java
	public class DemoSubscription implements Subscription {

		private final ExecutorService executorService = Executors.newSingleThreadExecutor();
		private Iterator<Integer> iter;
		private Subscriber subscriber;

		public DemoSubscription(Subscriber subscriber, Iterable<Integer> iter){
			this.subscriber = subscriber;
			this.iter = iter.iterator();
		}

		@Override
		public void request(long l) {
			System.out.println("[Subscription:request]" + l);
			executorService.execute(() -> {
				int i=0;
				if(!iter.hasNext()){
					subscriber.onComplete();
				} else {
					try {
						List<Integer> items = new ArrayList<>();
						while (i++ < l) {
							if (iter.hasNext()) {
								items.add(iter.next());
							} else {
								break;
							}
						}
						subscriber.onNext(items);
					} catch (Exception e) {
						subscriber.onError(e);
					}
				}
			});
		}

		@Override
		public void cancel() {
			System.out.println("[Subscription:cancel]");
			subscriber.onComplete();
		}
	}
	```
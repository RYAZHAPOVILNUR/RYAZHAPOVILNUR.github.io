---
description: 'Самая известная и широко используемая библиотека реактивного программирования на JavaScript - это RxJS...'
title: "Что нового в RxJS 7"
date: 2021-06-23T20:29:14+05:00
draft: false

---

## Что нового в RxJS 7

#### Самая известная и широко используемая библиотека реактивного программирования на JavaScript - это RxJS. Настолько, что количество загрузок в неделю сегодня составляет около 25 миллионов (2,3x React, 9,7x Angular, 11x Vue). Мы используем RxJS 6 3 года. RxJS 7, первая альфа-версия которого была выпущена в сентябре 2019 года, был выпущен шесть дней назад. Итак, какие изменения ждут нас в RxJS 7? В чем разница между RxJS 7 и 6? Стоит ли переходить на RxJS 7?

### TypeScript, размер, память и скорость в RxJS 7

### Улучшенные типы

#### Одно из основных изменений в RxJS 7, пожалуй, наименее заметное. Когда мы смотрим на журнал изменений, мы видим, что над типами проделано много работы. Особенно выделяются результаты по следующим темам:

- ### Функции, принимающие `n` параметры, такие как `of`, теперь могут правильно определять типы, даже если им передано более 8–9 параметров.

 ```
import { of } from "rxjs";

// Observable<string, number> (in RxJS 6 → No overload matches this call)
of(0, "A", 1, "B", 2, "C", 3, "D", 4, "E", 5, "F", 6, "G", 7, "H", 8, "I", 9, "...");
 ```

- ### Мы можем разделить группы, созданные с помощью, `groupBy` на их типы в соответствии с `key`.

```
import { of } from "rxjs";
import { groupBy, map, mergeMap } from "rxjs/operators";

// Observable<string, number>
of(0, "A", 1, "B", 2, "C", 3, "D", 4, "E", 5, "F", 6, "G", 7, "H", 8, "I", 9, "...")
  .pipe(
    groupBy((x): x is number => typeof x === "number"),
    mergeMap(group$ => (group$.key === true ? group$.pipe(map(String)) : group$))
  )
  .subscribe();
// Observable<string> (in RxJS 6 → Observable<string | number>)
```

- ### С помощью `filter(Boolean)` мы можем отфильтровать типы, которые не соответствуют условию.

```
import { of } from "rxjs";
import { filter } from "rxjs/operators";

// Observable<"" | 0 | null | undefined | Date>
of("" as const, 0 as const, null, undefined, new Date())
  .pipe(filter(Boolean))
  .subscribe();
// Observable<Date> (in RxJS 6 → Observable<unknown>)
```

- ### Теперь метод `next` учитывает тип `Subject`.

```
import { Subject } from "rxjs";

const number$ = new Subject<number>();
number$.next();

// Error: An argument for 'value' was not provided.
// in RxJS 6 → No error
```

### Меньший размер бандла

#### Бен Леш говорит, что, перебирая всю библиотеку шаг за шагом, они уменьшили общий размер пакета на 39% . Это большие усилия и достойный внимания результат. Итак, я попробовал это в небольшом проекте Angular. Результаты приведены ниже:

##### Таблица, показывающая размер пакета с RxJS 6
![](https://miro.medium.com/max/2400/1*tweVlLxR0Nt6Mh9iphUr4g.png)
###### размер пакета RxJS 6


##### Таблица, показывающая размер пакета с RxJS 7
![](https://miro.medium.com/max/2400/1*eFTkodeI0S2ca1flpkbL5Q.png)
###### размер пакета RxJS 7

#### Использование только RxJS 7 вместо RxJS 6, имело эффект на 10 КБ на общий размер пакета. Неплохо для небольшого приложения. Кроме того, мы видим, что это изменение больше повлияло на размер ленивого фрагмента, в котором я использовал много операторов.

### Меньше потребление памяти

#### В RxJS, тот `Observable`, который эмитит другой `Observable` - «observable высшего порядка».

```
import { of } from "rxjs";
import { map, concatAll } from "rxjs/operators";

of(1, 2, 3)
  .pipe(
    map(id =>
      fromFetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
        selector: resp => resp.json()
      })
    ),
    concatAll()
  )
  .subscribe(todo => console.log(todo.title));
// (after some time) delectus aut autem
// (after some time) quis ut nam facilis et officia qui
// (after some time) fugiat veniam minus
```

#### Этот `Observable`, что мы создали c `of`, называется «внешним observable», а то, что мы создали с `fromFetch` называется «внутренним observable». В RxJS 6 операторы также фиксируют значения, испускаемые внешними наблюдаемыми объектами. В RxJS 7 устранена необходимость в захвате внешних значений и снижено потребление памяти.

### Более быстрый RxJS

#### Честно говоря, сам не пробовал. Описания Бен Леша и то, что я слышал здесь и там, относятся к этому направлению. См. Твит ниже:

![](https://miro.medium.com/max/2400/1*Zf70uoDVGWWyEkm405cjkw.png)

#### Твит, в котором утверждается, что RxJS 7 примерно на 20% быстрее
#### Конечно, в твите не говорится, что «20%» - это фиксированное значение, рефакторинг и улучшения кодовой базы, должно быть, сработали.
> #### Возможно, вы видели это в твите, но не слишком волнуйтесь: поддержка для с `for await` тех пор была удалена.

### Примечательные особенности и изменения в RxJS 7

### toPromise → firstValueFrom, lastValueFrom

#### Начнем с изменения, о котором слышало большинство людей, использующих RxJS 6: `toPromise` теперь устарело. Но не бойтесь, он еще не удален из библиотеки. Таким образом, `toPromise` все еще можно использовать в RxJS 7. Однако я предлагаю вам убрать `toPromise` из своего проекта, потому что он будет скоро удален. Кроме того, для возвращаемого типа устанавливается значение `undefined`, что может нарушить работу проектов, использующих TypeScript в строгом режиме.

#### Так во что вы можете переформатировать это? Для начала вспомним, как работает `toPromise`:

```
import { interval } from "rxjs";
import { map, take } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

count1To5$.toPromise().then(console.log);
// (after ~5s) 5
```

#### Почему `5`, a не `1`? Это потому, что `toPromise` возвращает `Promise`, который резолвит с последнее значение, выпущенное при завершении источника `Observable`. В RxJS 7 вместо `toPromise`, мы можем использовать две разные функции: `firstValueFrom` и `lastValueFrom`.

```
import { interval, firstValueFrom, lastValueFrom } from "rxjs";
import { map, take } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

firstValueFrom(count1To5$).then(console.log);
// (after ~1s) 1

lastValueFrom(count1To5$).then(console.log);
// (after ~5s) 5
```

#### Если вам интересно, есть ли разница между `lastValueFrom` и `toPromise`, да, есть. А именно, `toPromise` вернет `undefined`, если источник `Observable` завершится без выдачи значения.

#### Это немного неверно, потому что:
- #### `Promise`, возвращаемый `toPromise` действительно может возвращать `undefined`. Невозможно отличить от состояния ошибки.
- #### Случай, когда значение не передается, следует рассматривать как ошибку.

#### Новая функция `lastValueFrom` выдает настраиваемую ошибку с формой `EmptyErrorImpl`, когда `Observable` завершается без выдачи значения. Обратите внимание на использование `catch` в примере ниже.

```
import { EMPTY, firstValueFrom } from "rxjs";

lastValueFrom(EMPTY).catch(console.log);
// (asynchronously) EmptyErrorImpl
```

#### Итак, как мы обнаруживаем эту ошибку? Мы будем использовать для этого класс `EmptyError`.
#### Не запутайтесь между `EmptyError` и `EmptyErrorImpl`. Следующая часть исходного кода объясняет, почему мы видим два разных имени для одной и той же ошибки:

```
export const EmptyError: EmptyErrorCtor = createErrorClass(
  _super =>
    function EmptyErrorImpl(this: any) {
      _super(this);
      this.name = "EmptyError";
      this.message = "no elements in sequence";
    }
);
```

#### Вы можете подумать, почему бы не сделать вот так:

```
export class EmptyError extends Error {
  constructor() {
    super("no elements in sequence");
    this.name = "EmptyError";
  }
}
```
#### Причина в том, что при `new EmptyError() instanceof EmptyError`, ожидается возврат выражения true, но когда TypeScript скомпилирован в ES5, он возвращает false. (См. TypeScript # 12123 ). Команда RxJS использовала `Object.create` для решения этой проблемы, и название `constructor` ошибки отличается от типа ошибки. Этот подход применяется ко всем настраиваемым ошибкам, выдаваемым RxJS.

> #### У них могло быть одно и то же имя, но эта фиксация остановила переопределение no-shadowed-variable правила TSLint и потребовала, чтобы имена были разными.

### combineLatest observable dictionary

#### В RxJS 6 мы могли передавать словари `Observable` в файлы `forkJoin`.

```
import { forkJoin, interval } from "rxjs";
import { map } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

forkJoin({ x: count1to5$, y: count6to9$ }).subscribe(console.log);
// (after ~5s) {x: 5, y: 9}
```

#### Теперь это также возможно с `combineLatest` в RxJS 7.

```
import { combineLatest, interval } from "rxjs";
import { map } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

combineLatest({ x: count1to5$, y: count6to9$ }).subscribe(console.log);
// (after ~1s) {x: 1, y: 6}
// (after ~1s) {x: 2, y: 6} (immediately) {x: 2, y: 7}
// (after ~1s) {x: 3, y: 7} (immediately) {x: 3, y: 8}
// (after ~1s) {x: 4, y: 8} (immediately) {x: 4, y: 9}
// (after ~1s) {x: 5, y: 9}
```
### combineLatestWith

#### Оператор `combineLatest` был устаревшим уже в RxJS 6.

```
import { interval } from "rxjs";
import { combineLatest, map } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(combineLatest(count6To9$)).subscribe(console.log);
// (after ~1s) [1,6]
// (after ~1s) [2,6] (immediately) [2,7]
// (after ~1s) [3,7] (immediately) [3,8]
// (after ~1s) [4,8] (immediately) [4,9]
// (after ~1s) [5,9]
```

#### В RxJS 7 его заменил оператор `combineLatestWith`.

### mergeWith

#### Оператор `merge` был устаревшим в RxJS 6.

```
import { interval } from "rxjs";
import { map, merge } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(merge(count6To9$)).subscribe(console.log);
// (after ~1s) 1 (immediately) 6
// (after ~1s) 2 (immediately) 7
// (after ~1s) 3 (immediately) 8
// (after ~1s) 4 (immediately) 9
// (after ~1s) 5
```

#### В RxJS 7 его заменил оператор `mergeWith`.

```
import { interval } from "rxjs";
import { map, mergeWith } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(mergeWith(count6To9$)).subscribe(console.log);
// (after ~1s) 1 (immediately) 6
// (after ~1s) 2 (immediately) 7
// (after ~1s) 3 (immediately) 8
// (after ~1s) 4 (immediately) 9
// (after ~1s) 5
```

### zipWith

#### Оператор `zip` был устаревшим в RxJS 6.

```
import { interval } from "rxjs";
import { map, zip } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(zip(count6To9$)).subscribe(console.log);
// (~1s apart) [1,6], [2,7], [3,8], [4,9]
```

#### В RxJS 7 его заменил оператор `zipWith`.

```
import { interval } from "rxjs";
import { map, zipWith } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(zipWith(count6To9$)).subscribe(console.log);
// (~1s apart) [1,6], [2,7], [3,8], [4,9]
```

### raceWith

#### Оператор `race` был устаревшим в RxJS 6.

```
import { interval } from "rxjs";
import { map, race } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(race(count6To9$)).subscribe(console.log);
// (~1s apart) 1, 2, 3, 4, 5
```

#### В RxJS 7 его заменил оператор `raceWith`.

```
import { interval } from "rxjs";
import { map, raceWith } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(raceWith(count6To9$)).subscribe(console.log);
// (~1s apart) 1, 2, 3, 4, 5
```

### concatWith

#### Оператор `concat` был устаревшим в RxJS 6.

```
import { interval } from "rxjs";
import { concat, map } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(concat(count6To9$)).subscribe(console.log);
// (~1s apart) 1, 2, 3, 4, 5, 6, 7, 8, 9
```

#### В RxJS 7 его заменил оператор `concatWith`.

```
import { interval } from "rxjs";
import { concatWith, map } from "rxjs/operators";

const count1To5$ = interval(1000).pipe(
  take(5),
  map(i => i + 1)
);

const count6To9$ = interval(1000).pipe(
  take(4),
  map(i => i + 6)
);

count1To5$.pipe(concatWith(count6To9$)).subscribe(console.log);
// (~1s apart) 1, 2, 3, 4, 5, 6, 7, 8, 9
```

### Более мощный timeout

#### До RxJS 7 мы могли сделать следующее, чтобы установить отдельный тайм-аут для начального значения, которое будет выдавать поток:

```
import { concat, partition, timer } from "rxjs";
import { first, share, timeout } from "rxjs/operators";

const count$ = timer(3000, 2000).pipe(share());

const [first$, rest$] = partition(count$, (_, index) => index === 0);

concat(
  first$.pipe(
    timeout(5000),
    first()
  ),
  rest$.pipe(timeout(1000))
).subscribe({ next: console.log, error: console.error });
// (after ~3s) 0
// (after ~1s) Error: Timeout has occurred
```

#### Честно говоря, описанная выше последовательность управления была слишком сложной для такой простой вещи. С помощью RxJS 7 мы теперь можем передать объект конфигурации `timeout` оператору:

```
import { timer } from "rxjs";
import { timeout } from "rxjs/operators";

const count$ = timer(3000, 2000);

count$
  .pipe(timeout({ first: 5000, each: 1000 }))
  .subscribe({ next: console.log, error: console.error });
// (after ~3s) 0
// (after ~1s) Error: Timeout has occurred
```

#### Кроме того, в объекте конфигурации `timeoutWith`, оператор устарел и заменен `with` параметром. В RxJS 6 мы делали следующее:

```
import { timer } from "rxjs";
import { timeoutWith } from "rxjs/operators";

timer(5000, 1000)
  .pipe(timeoutWith(2000, of("timeout")))
  .subscribe(console.log);
// (after ~2s) timeout
```

В RxJS 7 мы делаем это:

```
import { timer } from "rxjs";
import { timeout } from "rxjs/operators";

timer(5000, 1000)
  .pipe(timeout({ first: 2000, with: _ => of("timeout") }))
  .subscribe(console.log);
// (after ~2s) timeout
```

### Параметр resetOnSuccess для оператора повтора

#### В RxJS 6 `retry` в качестве параметра будет приниматься только количество повторных попыток.

```
import { defer, from } from "rxjs";
import { retry, tap } from "rxjs/operators";

const values = ["_", 0, 1, 0, 2, 0, 3, 0, 0, 0, 4];

defer(() => {
  values.shift();
  return from(values);
})
  .pipe(
    tap(i => {
      if (!i) throw "ERROR";
    }),
    retry(2)
  )
  .subscribe({ next: console.log, error: console.warn });
// (synchronously) 1, ERROR
```

#### Это число не сбрасывается после успешных попыток. Теперь, в RxJS 7 мы можем передать оператору `retry`, параметр с именем `resetOnSuccess`.

```
import { defer, from } from "rxjs";
import { retry, tap } from "rxjs/operators";

const values = ["_", 0, 1, 0, 2, 0, 3, 0, 0, 0, 4];

defer(() => {
  values.shift();
  return from(values);
})
  .pipe(
    tap(i => {
      if (!i) throw "ERROR";
    }),
    retry({ count: 2, resetOnSuccess: true })
  )
  .subscribe({ next: console.log, error: console.warn });
// (synchronously) 1, 2, 3, ERROR
```


### Конфигурация Share

#### В RxJS 7, оператор `share` получил довольно обширный объект конфигурации. Теперь, похоже, что больше нет необходимости в `shareReplay`. Вспомним, как `shareReplay` работал в RxJS 6:

```
import { interval, of, zip } from "rxjs";
import { map, shareReplay } from "rxjs/operators";

const shared$ = zip(interval(1000), of("A", "B", "C", "D", "E")).pipe(
  map(([, char]) => char),
  shareReplay({ refCount: true, bufferSize: 3 })
);

shared$.subscribe(console.log);
// (~1s apart) A, B, C, D, E

setTimeout(() => shared$.subscribe(console.log), 6000);
// (after ~6s, all at once) C, D, E
```

#### Посмотрим, как это сделать с новой конфигурацией `share`.

```
import { interval, of, ReplaySubject, zip } from "rxjs";
import { map, share } from "rxjs/operators";

const shared$ = zip(interval(1000), of("A", "B", "C", "D", "E")).pipe(
  map(([, char]) => char),
  share({
    connector: () => new ReplaySubject(3),
    resetOnComplete: false,
    resetOnError: false,
    resetOnRefCountZero: false
  })
);

shared$.subscribe(console.log);
// (~1s apart) A, B, C, D, E

setTimeout(() => shared$.subscribe(console.log), 6000);
// (after ~6s, all at once) C, D, E
```

#### Свойства, которые начинаются со «сброса» (reset) в объекте конфигурации, позволяют `Observable` сбросить их, и снова становятся «холодными» при наступлении соответствующего состояния. Например, `resetOnRefCountZero` сбрасывает ресурс, если количество подключенных `Observers` снова становится равным нулю из-за `unsubscribe`.

### Сonnect и Сonnectable

#### В RxJS 7 появился новый оператор `connect` для мультикаста источника `Observable`. Работает это так:

```
import { merge, of } from "rxjs";
import { connect, filter, map } from "rxjs/operators";

const chars$ = of("A", "b", "C", "D", "e", "f", "G");

chars$
  .pipe(
    connect(shared$ =>
      merge(
        shared$.pipe(
          filter(x => x.toLowerCase() === x),
          map(x => `lower ${x.toUpperCase()}`)
        ),
        shared$.pipe(
          filter(x => x.toLowerCase() !== x),
          map(x => `upper ${x}`)
        )
      )
    )
  )
  .subscribe(console.log);
// (synchronously) upper A
// (synchronously) lower B
// (synchronously) upper C
// (synchronously) upper D
// (synchronously) lower E
// (synchronously) lower F
// (synchronously) upper G
```

#### Также есть новая функция с именем `connectable`, которая возвращает объекты `ConnectableObservableLike`. Мы могли бы повторить приведенный выше пример следующим образом:

```
import { connectable, merge, of } from "rxjs";
import { filter, map } from "rxjs/operators";

const chars$ = of("A", "b", "C", "D", "e", "f", "G");
const connectableChars$ = connectable(chars$);

const lower$ = connectableChars$.pipe(
  filter(x => x.toLowerCase() === x),
  map(x => `lower ${x.toUpperCase()}`)
);

const upper$ = connectableChars$.pipe(
  filter(x => x.toLowerCase() !== x),
  map(x => `upper ${x}`)
);

merge(lower$, upper$).subscribe(console.log);

connectableChars$.connect();
// (synchronously) upper A
// (synchronously) lower B
// (synchronously) upper C
// (synchronously) upper D
// (synchronously) lower E
// (synchronously) lower F
// (synchronously) upper G
```

#### При модификации на последнем подключенном элементе, `connectable` начал получать объект конфигурации с `connector` и `resetOnDisconnect` свойствами. Посмотрите, как мы отключаемся и снова подключаемся в примере ниже.

```
import { connectable, interval, merge, of, Subject, zip } from "rxjs";
import { filter, map } from "rxjs/operators";

const chars$ = zip(interval(1000), of("A", "b", "C", "D", "e", "f", "G")).pipe(
  map(([, char]) => char)
);
const connectableChars$ = connectable(chars$, {
  connector: () => new Subject(),
  resetOnDisconnect: true
});

const lower$ = connectableChars$.pipe(
  filter(x => x.toLowerCase() === x),
  map(x => `lower ${x.toUpperCase()}`)
);

const upper$ = connectableChars$.pipe(
  filter(x => x.toLowerCase() !== x),
  map(x => `upper ${x}`)
);

function connect() {
  merge(lower$, upper$).subscribe(console.log);
  return connectableChars$.connect();
}

const connection = connect();
setTimeout(() => {
  connection.unsubscribe();
  connect();
}, 3000);
// (after ~1s) upper A
// (after ~1s) lower B
// (after ~1s) upper C
// (after ~1s) upper A
// (after ~1s) lower B
// (after ~1s) upper C
// (after ~1s) upper D
// (after ~1s) lower E
// (after ~1s) lower F
// (after ~1s) upper G
```

#### Теперь, `connectable`, `connect` и `share` являются достаточными для мультикастинга, а `multicast`, `publish`, `publishBehavior`, и `publishLast` операторы устарели.

### Aнимация (animationFrames)

#### В RxJS 7, есть новый `observable`, который оборачивает `requestAnimationFrame` и `cancelAnimationFrame` функции: `animationFrames`. Как следует из названия, он используется для создания анимации.

```
import { animationFrames, combineLatest, concat } from "rxjs";
import { endWith, map, takeWhile } from "rxjs/operators";

const h1 = document.querySelector("h1")!;

combineLatest({
  x: tween(0, 200, 3600),
  y: wave(25, 1200, 3)
}).subscribe(({ x, y }) => {
  h1.style.transform = `translate3d(${x}px, ${y}px, 0)`;
});

function tween(start: number, end: number, duration: number) {
  const delta = end - start;

  return animationFrames().pipe(
    map(({ elapsed }) => elapsed / duration),
    takeWhile(percentage => percentage < 1),
    endWith(1),
    map(percentage => percentage * delta + start)
  );
}

function wave(length: number, duration: number, repeat = 1) {
  const tweens = [
    tween(0, length, duration / 4),
    tween(length, -length, duration / 2),
    tween(-length, 0, duration / 4)
  ];

  const animation = Array(repeat)
    .fill(tweens)
    .flat();

  return concat(...animation);
}
```

#### Простая анимация, созданная приведенным выше кодом, выглядит следующим образом:

![](https://miro.medium.com/max/2400/1*b1yn2ZJmQxv4-KJXyNIfYQ.gif)
##### RxJS 7 animationFrames, перемещающие заголовок на странице

### config.onUnhandledError
#### В RxJS 7, объект `config` содержит асинхронный catcher для необработанных ошибок: `onUnhandledError`.

```
import { config, throwError } from "rxjs";

config.onUnhandledError = console.warn;
throwError(() => "TEST ERROR 1").subscribe();
throwError(() => "TEST ERROR 2").subscribe({ error: console.warn });
// (synchronously) TEST ERROR 2
// (asynchronously) TEST ERROR 1
```

### Пользовательские ошибки RxJS возвращают стек вызовов

#### В начале статьи мы упоминали, что RxJS использует `Object.create` для пользовательских ошибок. Фактически, `Object.setPrototypeOf` использовался раньше, но при разработке v6.3.0 от него отказались для поддержки IE10. В октябре 2018 года было создано issue, где говорилось, что пользовательские ошибки теряют свой стек вызовов. Наконец, с 7.0.0-beta.5 стек вызовов был снова добавлен к ошибкам.

```
import { EMPTY } from "rxjs";
import { first } from "rxjs/operators";

EMPTY.pipe(first()).subscribe({ error: console.warn });
// (synchronously) EmptyErrorImpl
// v6.6.7 has no call stack, but v7 does
```

### Заключение

#### В RxJS 7 есть еще много изменений, но для полного обзора не хватило бы сил ни у меня, ни у вас. Я попытался собрать в этой статье то, что посчитал важным. Возможно, переход на 7 версию будет не таким драматичным, как переход с 4 на 5 или с 5 на 6, но я должен отметить, что RxJS теперь является стабильной библиотекой.
#### В RxJS 7 все еще много устаревшего, но, чтобы упростить обновление, разработчики ничего не удалили. Это отличная новость. Тем не менее, пользователи TypeScript иногда могут испытывать трудности. Я думаю, что имеет смысл перейти на RxJS 7 для текущих проектов. Я сделаю это для своих проектов при первой возможности, и вам тоже рекомендую.
### Надеюсь, это было полезно. Увидимся в другой статье.

Перевод статьи: https://medium.com/volosoft/whats-new-in-rxjs-7-a11cc564c6c0

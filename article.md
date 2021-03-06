Но прежде немного о себе.
За время своей работы я перепробовал много всякой наркоты:
Vanilla
Backbone,
React,
Полный БЭМ-стек,
Polymer,
Angular2

Я перевидал кучу подходов, технологий и знаете что,
пора заканчивать с фреймворковым расизмом.

Фреймворки наконец-то созрели до того момента,
что стало неважным, что ты используешь. Важно как.


Среди программистов часто бывают модные крестовые походы.
Firefox vs IE.
Linux vs Windows.
Сегодня я расскажу об еще одном из холиваров - Angular vs React.

Однако с несколько необычной точки зрения.


Внутри Angular и React лежат одни и те же идеи. Но вот их реализация весьма
различается. Давайте попробуем разобрать, что лежит внутри одного и другого,
а потом все-таки попробовать привести их к общему знаменателю.


Я сам пропесочил немало статей про Angular, плюс и на работе было немало обсуждений с коллегами.
И я для себя выделил несколько различий. Мы попробуем их разобрать

1. Шаблонизация.
Angular 2 - можем писать шаблоны в отдельном файле и они имеют свой синтаксис.
```html
<div class="widget-list">
	<someWidget
		[ngClass]="{isBlue: true}"
		*ngFor="let model of models"
		*ngIf="isLoaded"
		[model]="model"
		(close)="onClose($event, model)"
	></someWidget>
</div>
```

Вы спросите, как Angular понимает как и где какой виджет применять:
```dart
@Component(
	directives: [
		CORE_DIRECTIVES,
		SomeWidgetClass
	]
)
class WidgetList {
	Model model;
	bool isLoaded;
	onClose(e, model) {}
	// ...
}
```
Расширять функционал шаблонов мы можем с помощью директив.
Если вкратце - мы пишем директиву, на нее матчим селектор.
Если DI ангуляра содержит эту директиву в зависимостях вьюхи - он ищет селектор, если находит
применяет код директивы.
А уж сам код директивы - зависит от вашей фантазии.

Это довольно мощный функционал. Подробнее о нем, можно рассказывать целый митап.
Примеры директив:
ngClass, ngFor и т.д.

React:
JSX.
```js
class SomeWidget {
	onClose(e, model) {}
	render() {
		var widgets = models.map((model) => {
			return isLoaded ? ( <someWidget
					className="{{cn({isBlue: true})}}"
					model="model"
					close="onClose.bind(this, model)"
				></someWidget> ) : null;
		}): return (
			<div class="widget-list">
				{{widgets}}
			</div>
		);
	}
```


Функционал ваших шаблонов ограничен только вашей фантазией.
Шаблоны обычный Javascript

С помощью дополнительных модулей можно раскрутить любой функционал.
Здесь например модуль cn.
cn просто дополнительный модуль на js, которому передаешь хэш с параметрами
- на выходе строка для класса

С помощью дополнительных же модулей, можно и вынести шаблон в отдельный файл.

2. ChangeDetection.
Angular 2.
 реализован на основе zone.js

Вкратце позволяет получить всегда корректный контекст this даже в асинхронных операциях
Если контекст изменяется - zone.js создает новую зону и следит за всеми изменениями, которые
происходят внутри.

Вы можете попробовать реализовать свою стратегию для Angular 2, но хочу вас предупредить это дело довольно нетривиальное. На это я думаю нужно выделять отдельную тему на митапе. Скажу только, что в ангуляре есть несколько уже реализованных стратегий по умолчанию и вы можете использовать их. Для большинства кейсов вам их хватит с головой.


React.
virtual dom на основе сравнений инстансов отрисовываемых классов.
Можно управлять через
shouldComponentUpdate -
обычная функция внутри виджета, принимающая на вход новые и старые props и state, сравниваете их между собой
И возвращаете true, false.

3. Композиция виджетов.
Этот пункт во многом перекликается с Шаблонизацией, но я вынес его в отдельный пункт.

Angular 2
Есть два способа осуществлять композицию виджетов в Angular:
1. Структурные директивы
```dart
@Directive(selector: '[myUnless]')
class UnlessDirective {
  TemplateRef _templateRef;
  ViewContainerRef _viewContainer;
  UnlessDirective(this._templateRef, this._viewContainer);
  @Input()
  set myUnless(bool condition) {
    if (!condition) {
      _viewContainer.createEmbeddedView(_templateRef);
    } else {
      _viewContainer.clear();
    }
  }
}
```

```html
<p *myUnless="!condition">
  condition is true and myUnless is false.
</p>
```

Получают доступ к шаблону который нужно отрендерить и с помощью API ViewContainer - производим рендеринг.
Выполняем свой ChangeDetection и т.д.

2. <ng-content></ng-content>
Еще изменить шаблон, который мы будем рендерить можно и с помощью ng-content.

```html
<Grid>
	<Cell></Cell>
	<Header></Header>
</Grid>
```

```html
<div>
	<ng-content></ng-content>
</div>
```

React.
Предоставляет два основных способа:
1. Передача виджетов через props.
```html
<Grid
	cellRenderer={{() => <Cell />}}
	header={{<Header />}}
></Grid>
```
2. Передача виджета через children.
```html
<Grid>
	<Cell></Cell>
	<Header></Header>
</Grid>
```

Мне больше нравится как сделана композиция виджетов в реакте. Достаточно просто и быстро
реализовать ее самостоятельно.

4. IoC

Так поднимите руки, кто знает что означают эти три буквы?

Набор принципов для написания поддерживаемого кода. Суть в том, что код должен делиться на компоненты.
И они в свою очередь должны по минимуму зависеть друг от друга.

React:
Обычнейший service locator,
Т.е. глобальный контекст, в котором регистрируется кто хочет и по определенному ключу достает что хочет.

```js
class Message {
	contextTypes: {
		color: React.PropTypes.string
	}
	render() {
		return <div>{{this.context.color}}</div>
	}
}
class MessageList {
	childContextTypes: {
		color: React.PropTypes.string
	}
	getChildContext() {
		return {color: "purple"};
	}
	render() {
		return <Message />
	}
}
```

Angular 2.
Реализован полноценный DI.
DI = Service Locator + Управление зависимостями.
Который позволяет инжектить как компоненты, так и сервисы с данными.

```dart
@Component(
	providers: const [ColorService],
	directives: const [Message],
	template: '''
	<Message />
'''
)
class MessageList {
	ColorService colorService;
	MessageList(this.colorService) {
		colorService.color = 'purple';
	};
}

@Component(
	template: '''
	<div>{{this.context.color}}</div>
'''
)
class Message {
	ColorService colorService;
	Message(this.colorService);
}
```

5. Входные данные.

React.
Входные данные - props. Через них мы можем передавать:
1. виджеты.
2. данные.
3. коллбеки, пробрасываемые как данные

Angular.
Разделяем входные и выходные данные:
1. Виджетами пробрасываемыми через ng-content.
2. входные данные @Input
3. выходные события @Output



Если сравнивать Angular и React, то на ум приходит другой извечный холивар - IDE.

Angular - Intellij Idea, т.е. в коробке есть почти все что нужно, но есть некоторые
моменты, которые хочется подпилить под себя.
И это не всегда удобно делается.

React - Atom. Из коробки предоставлен базовый функционал. Все остальное наруливается устанавливается
внешними плагинами.

Angular

На мой взгляд если у вас большая команда > 10 человек
то подход Angular вам подойдет больше.
Как правило бизнес-процессы в таких командах уже отлажены,
Основная задача - быстро встроится и сразу приносить пользу для бизнеса.
Выбор в принципах построении архитектуры уже сделан.
Большая часть утилитного API уже реализована. Бери и пользуйся.

React
React больше подходит небольшим командам с энтузиастом-архитектором от веб-разработки,
который следит за последними трендами.
Такие энтузиасты должны подготовить масштабируемую архитектуру.
Подобрать нужные утильные модули.
Написать необходимую обвязку для интеграции.
Основная задача - быстрая адаптация под любые будущие требования бизнеса.

В общем-то мы рассмотрели основные особенности работы с Angular и React.
На мой взгляд пора перестать вести религиозные войны, а вместо этого писать модули, которые с небольшой обвязкой можно будет использовать как в Angular, так и в React. Т.к. общие подходы и паттерны и у того, и другого схожи.

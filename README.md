# Мощная библиотека Redux-Form

Хотелось бы начать, вот с такого ньюанса. Существуют такие разработчики, уровень которых не позволяет делать дельные советы в выборе той или иной библиотеки, что видно в различных дискуссиях. Вот с таким я столкнулся, когда для работы над проектом выбрал `Redux-React`.

# Поехали

Начнем с примера и задач, которые нужно было решать не используя `Redux-React`.

# Задачи

 1. Сделать счетчик, который бы имел минимальное и максимальное значения
 2. Влиять на контролы увеличения и уменьшения значения счетчика
 3. Выводить суммарное число, для нескольких счетчиков
 4. Один из счетчиков зависит от другого счетчика (например: нельзя увеличить значение)

# Пример без исользования `Redux-React`

У нас есть счетчик `parent`, который имеет минимальное значение "1" и максимальное "9". Имеется счетчик `children`, который 
зависит от `parent` и имеет минимальное значение "0" и максимальное значение меньше максимального значения счетчика `parent`. Если условия для счетчик не выполняется, то не выводим кнопки контролы.

```javascript
render() {
   	const {reducerCounter: {parent, children, total}} = this.props

	return (
		<div>
			<div><strong>parents</strong></div>
			{ (parent > 1) &&
				<button onClick={this.decrement.bind(this, 'parent')}>-</button>
			}
			<span>
				{parent}
			</span>
			{ (parent < 9) &&
			  <button onClick={this.increment.bind(this, 'parent')}>+</button>
			}
			<div><strong>childrens</strong></div>
			{ (children > 0) &&
				<button onClick={this.decrement.bind(this, 'children')}>-</button>
			}
			<span>
				{children}
			</span>
			{ (parent > children) &&
			  <button onClick={this.increment.bind(this, 'children')}>+</button>
			}
			<p>total: {total}</p>
		</div>
	)
}
```

#### Диспетчеры для изменения значений счетчиков (например: диспетчер увеличения значения счетчика)
```javascript
	export function increment(type) {
	return {
		type: 'Increment',
		amount: type
	}
}
```

#### Редьюсер, который управляет состоянием
```javascript
	export function reducerCounter(state=initialState, action) {
	  let typeObj;
	  const type = action.amount;
	  if (type === 'parent') typeObj = state.parent;
	  if (type === 'children') typeObj = state.children;
	  switch (action.type) {
		case 'Increment':
		  state[type] = typeObj + 1
		  state.total += 1
		  return {...state};
		case 'Decrement':
		  state[type] = typeObj - 1
		  state.total -= 1
		  return {...state};
		default:
		  return state;
	  }
  }
```
#### Выводы:

1.	Компонент счетчиков не универсальный, т.е. если мы захотим еще счетчик необходимо явно его задавать
2.	Если нужно будет изменять данные по другому, то придется добовлять диспетчеры и редьюсер


# Пример c исользованием `Redux-React`

#### Форма для счетчиков
```javascript
 render() {
    const {parent, total} = this.props

    return (
      <div>
        <Field
          name="parent"
          component={Counter}
        />
        <Field
          name="children"
          component={Counter}
          parent={parent}
        />

        <p>Total: {total}</p>
      </div>
    )
 }
```

#### Компонент `Counter` для использования в <Field/>
```javascript
render() {
    const {input: { value, name }, parent} = this.props;

    return (
      <div>
        <div><strong>{name}</strong></div>
        { ((name === 'parent' && value > 1) || (name !== 'parent' && value !== 0)) &&
          <button onClick={this.decrement}>-</button>
        }
        <span>
          {value}
        </span>
        { ((name === 'children' && value < parent) || (name !== 'children' && value !== 9)) &&
          <button onClick={this.increment}>+</button>
        }
      </div>
    )
  }
```

#### Редьюсер
```javascript
form: formReducer.plugin({
	counters: (state, action) => {
	  switch(action.type) {
		case '@@redux-form/CHANGE':
		  const {parent, children, infant} = state.values
		  state.values.total = parent + children + infant
		return { ...state}
		default:
		  return state
	  }
	}
})
```

#### Получаем значения полей в `redux-form`, чтобы вывести значения `total` и `parent`
```javascript
	const selector = formValueSelector('counters')

	const mapStateToProps = (state) => {
	  const parent = selector(state, 'parent')
	  const total = selector(state, 'total')

	  return {
		 parent,
		 total
	  }
	}
```

#### Выводы:

1. Получили легко расширяемый компонент для счетчиков
2. Код самой формы выглядит очень просто, т.к. весь код лежит в компоненте `{Counter}`
3. Компонеты формы меняют свое состояние без использования диспетчеров

# Что я для себя выяснил

 1. Изменения в состоянии хранилища проводить в `reducer.plugin()`. Функция `plugin` позволяет подписаться на события в форме.
 2. Для получения нужных `props` в <Field/>(когда мы по каким-то причинам используем вне `render()` компонента) или "самописном . Просто оборачиваем его в <FormSection name="">

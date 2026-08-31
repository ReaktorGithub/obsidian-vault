
const [value, setValue] = useState(calculateValue());
`calculateValue()` будет вызываться **при каждом рендере компонента**, хотя результат нужен только при первоначальной инициализации.

const [value, setValue] = useState(() => calculateValue());

React вызовет функцию для получения начального значения **только при инициализации state**.


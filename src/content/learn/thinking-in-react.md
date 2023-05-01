---
title: Pensando em React
---

<Intro>

React pode mudar como você pensa nos designs que você vê e nos aplicativos que você constrói. Quando você constrói uma interface de usuário com o React, você primeiro quebrará em partes chamadas *componentes*. Em seguida, você descreverá os diferentes estados para cada um de seus componentes. Por fim, você conectará seus componentes para que os dados fluam por eles. Neste tutorial, vamos guiá-lo pelo processo de pensamento de construção de uma tabela de dados de produtos pesquisável com React.

</Intro>

## Comece com o Mock {/*start-with-the-mockup*/}

Pense que você já tem uma API JSON e um mock de um designer.

A API JSON retorna alguns dados que parecem com isso:

```json
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

O mock se parece com isso:

<img src="/images/docs/s_thinking-in-react_ui.png" width="300" style={{margin: '0 auto'}} />

Para implementar a UI em React, você normalmente irá seguir os mesmos cinco passos.

## Passo 1: Separe a UI em uma hierarquia de componentes {/*step-1-break-the-ui-into-a-component-hierarchy*/}

Comece desenhando retângulos ao redor de cada componente e subcomponente no mock e nomeando-os. Se estiver trabalhando com designers, eles podem já ter nomeado esses componentes em suas ferramentas de design. Pergunte a eles!

Dependendo do seu conhecimento, você pode pensar como separar o design em componentes de diferentes maneiras:

* **Programação**--use as mesmas técnicas para decidir se você deve criar uma nova função ou objeto. Uma dessas técnicas é o [princípio da responsabilidade única](https://en.wikipedia.org/wiki/Single_responsibility_principle), ou seja, um componente deve, idealmente, fazer uma única coisa. Se ele acabar crescendo, deve ser decomposto em subcomponentes menores. 
* **CSS**--considere o que você faria com os seletores de classe. (No entanto, os componentes são um pouco menos granulares.)
* **Design**--considere como você organizaria as camadas do design.

Se o seu JSON estiver bem estruturado, você geralmente encontrará que ele se encaixa naturalmente na estrutura de componentes da sua UI. Isso acontece porque a UI e os modelos de dados geralmente têm a mesma arquitetura de informações--ou seja, o mesmo formato. Separe sua UI em componentes, onde cada componente corresponde a uma parte do seu modelo de dados.

Existem cinco componentes nesta tela:

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1. `FilterableProductTable` (cinza) contém a aplicação inteira.
2. `SearchBar` (azul) recebe o input do usuário.
3. `ProductTable` (lavanda) Exibe e filtra a lista de acordo com o input do usuário.
4. `ProductCategoryRow` (verde) Exibe um cabeçalho para cada categoria.
5. `ProductRow`	(amarelo) exibe uma linha para cada produto.

</CodeDiagram>

</FullWidth>

Se você olhar para `ProductTable` (lavanda), verá que o cabeçalho da tabela (contendo os rótulos "Name" e "Price") não é seu próprio componente. Isso é uma questão de preferência e você pode ir para qualquer direção. Para este exemplo, é parte de `ProductTable` porque aparece dentro da lista de `ProductTable`. No entanto, se esse cabeçalho crescer para ser complexo (por exemplo, se você adicionar ordenação), você pode movê-lo para seu próprio componente `ProductTableHeader`.

Agora que você identificou os componentes no mock, organize-os em uma hierarquia. Os componentes que aparecem dentro de outro componente no mock devem aparecer como um filho na hierarquia:

* `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
        * `ProductCategoryRow`
        * `ProductRow`

## Passo 2: Construa uma versão estática em React {/*step-2-build-a-static-version-in-react*/}

Agora que você tem uma hierarquia de componentes, é hora de implementar o seu app. O modo mais direto é construir uma versão que renderiza a UI do seu modelo de dados sem adicionar nenhuma interatividade... ainda! É mais fácil construir a versão estática primeiro e adicionar interatividade depois. Construir uma versão estática requer muita digitação e pouco pensamento, mas adicionar interatividade requer muito pensamento e não muita digitação.

Para construir uma versão estática do seu app que renderize o seu modelo de dados, você irá querer construir [componentes](/learn/your-first-component) que reutilizem outros componentes e passar dados usando [props.](/learn/passing-props-to-a-component) Props são uma maneira de passar dados do pai para o filho. [Se você está familiarizado com o conceito de [state](/learn/state-a-components-memory) (estado), não use state para construir essa versão estática. Estado é reservado apenas para interatividade, ou seja, dados que mudam ao longo do tempo. Como esta é uma versão estática do app, você não precisa dele.]

Você pode construir tando de "cima para baixo", começando com a construção dos componentes mais altos na hierarquia (como `FilterableProductTable`), ou de "baixo para cima", trabalhando a partir dos componentes mais baixos (como `ProductRow`). Em exemplos mais simples, geralmente é mais fácil ir de cima para baixo, e em projetos maiores, é mais fácil ir de baixo para cima.

<Sandpack>

```jsx App.js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Mostrar apenas produtos em estoque
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 10px;
}
td {
  padding: 2px;
  padding-right: 40px;
}
```

</Sandpack>

(Se esse código parecer intimidador, dê uma olhada no [Início Rápido](/learn/) primeiro!)

Depois de construir seus componentes, você terá uma biblioteca de componentes reutilizáveis que renderizam seu modelo de dados. Por esse ser um app estático, os componentes retornarão apenas JSX. O componente no topo da hierarquia (`FilterableProductTable`) receberá seu modelo de dados como uma prop. Isso é chamado de _fluxo de dados unidirecional_ porque os dados fluem do componente de nível superior para os componentes na parte inferior da árvore.

<Pitfall>

A esta altura, você não deve estar usando nenhum valor de state. Isso é para o próximo passo!

</Pitfall>

## Passo 3: Identifique a representação mínima, mas completa do estado da UI {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

Para deixar a UI interativa, você precisa permitir que os usuários alterem seu modelo de dados subjacente. Você usará *state* para isso.

Pense no estado como o conjunto mínimo de dados que sua aplicação precisa lembrar. O princípio mais importante para estruturar o estado é mantê-lo [DRY: Don't Repeat Yourself (Não repita a si mesmo).](https://pt.wikipedia.org/wiki/Don%27t_repeat_yourself) Descubra a representação mínima absoluta do estado que sua aplicação precisa e calcule tudo o mais sob demanda. Por exemplo, se você está construindo uma lista de compras, você pode armazenar os itens como um array no estado. Se você também quiser exibir o número de itens na lista, não armazene o número de itens como outro valor de estado - em vez disso, leia o comprimento do seu array.

Agora pense sobre todas as peças de dados neste exemplo de aplicação:

1. A lista original de produtos
2. O texto de busca que o usuário digitou
3. O valor da checkbox
4. A lista filtrada de produtos

Qual desses são state? Identifique os que não são:

* **Permanece inalterado** ao longo do tempo? Se sim, não é state.
* É **passado de um pai** via props? Se sim, não é state.
* **Você pode computá-lo** com base no estado ou props existentes em seu componente? Se sim, *definitivamente* não é state!

O que sobrou provavelmente é state.

Vamos revisá-los um por um novamente:

1. A lista original de produtos é **passada como props, então não é state.**
2. O texto de busca parece ser state, pois mudará ao longo do tempo e não pode ser computado a partir de nada.
3. O valor da checkbox parece ser state, pois mudará ao longo do tempo e não pode ser computado a partir de nada.
4. A lista filtrada de produtos **não é state porque pode ser computada** pegando a lista original de produtos e filtrando-a de acordo com o texto de busca e o valor da checkbox.	

Isso significa que apenas o texto de busca e o valor da checkbox são state! Muito bem!

<DeepDive>

#### Props vs State {/*props-vs-state*/}

Existem dois tipos de "modelos" de dados no React: props e state. Os dois são muito diferentes:

* [**Props** são como argumentos que você passa](/learn/passing-props-to-a-component) para uma função. 
Eles deixão o componente pai passar dados para o componente filho e customizar sua aparência. Por exemplo, um `Form` pode passar uma prop `color` para um `Button`.
* [**State** é como a memória de um componente.](/learn/state-a-components-memory) Ele permite que um componente acompanhe algumas informações e as altere em resposta a interações. Por exemplo, um `Button` pode acompanhar o estado `isHovered`.

Props e state são diferentes, mas eles trabalham juntos. Um componente pai frequentemente manterá algumas informações no estado (para que possa alterá-lo) e *passá-lo* para componentes filhos como suas props. Tudo bem se a diferença ainda parecer confusa na primeira leitura. Leva um pouco de prática para realmente grudar!

</DeepDive>

## Passo 4: Identifique onde seu state deve viver {/*step-4-identify-where-your-state-should-live*/}

Depois de identificar os dados mínimos de estado do seu aplicativo, você precisa identificar qual componente é responsável por alterar esse estado, ou *possui* o estado. Lembre-se: o React usa fluxo de dados unidirecional, passando dados da hierarquia de componentes do componente pai para o filho. Pode não ser imediatamente claro qual componente deve possuir qual estado. Isso pode ser desafiador se você é novo neste conceito, mas você pode descobrir seguindo estas etapas!

Para cada pedaço de state em sua aplicação:

1. Identifique *cada* componente que renderize algo baseado em seu state. 
2. Encontre o componente pai comum mais próximo-- um componente acima de todos na hierarquia.
3. Decide where the state should live:
    1. Often, you can put the state directly into their common parent.
    2. You can also put the state into some component above their common parent.
    3. If you can't find a component where it makes sense to own the state, create a new component solely for holding the state and add it somewhere in the hierarchy above the common parent component.

In the previous step, you found two pieces of state in this application: the search input text, and the value of the checkbox. In this example, they always appear together, so it makes sense to put them into the same place.

Now let's run through our strategy for them:

1. **Identify components that use state:**
    * `ProductTable` needs to filter the product list based on that state (search text and checkbox value). 
    * `SearchBar` needs to display that state (search text and checkbox value).
1. **Find their common parent:** The first parent component both components share is `FilterableProductTable`.
2. **Decide where the state lives**: We'll keep the filter text and checked state values in `FilterableProductTable`.

So the state values will live in `FilterableProductTable`. 

Add state to the component with the [`useState()` Hook.](/reference/react/useState) Hooks are special functions that let you "hook into" React. Add two state variables at the top of `FilterableProductTable` and specify their initial state:

```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);  
```

Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as props:

```js
<div>
  <SearchBar 
    filterText={filterText} 
    inStockOnly={inStockOnly} />
  <ProductTable 
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

You can start seeing how your application will behave. Edit the `filterText` initial value from `useState('')` to `useState('fruit')` in the sandbox code below. You'll see both the search input text and the table update:

<Sandpack>

```jsx App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} />
      <ProductTable 
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 5px;
}
td {
  padding: 2px;
}
```

</Sandpack>

Notice that editing the form doesn't work yet. There is a console error in the sandbox above explaining why:

<ConsoleBlock level="error">

You provided a \`value\` prop to a form field without an \`onChange\` handler. This will render a read-only field.

</ConsoleBlock>

In the sandbox above, `ProductTable` and `SearchBar` read the `filterText` and `inStockOnly` props to render the table, the input, and the checkbox. For example, here is how `SearchBar` populates the input value:

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
```

However, you haven't added any code to respond to the user actions like typing yet. This will be your final step.


## Step 5: Add inverse data flow {/*step-5-add-inverse-data-flow*/}

Currently your app renders correctly with props and state flowing down the hierarchy. But to change the state according to user input, you will need to support data flowing the other way: the form components deep in the hierarchy need to update the state in `FilterableProductTable`. 

React makes this data flow explicit, but it requires a little more typing than two-way data binding. If you try to type or check the box in the example above, you'll see that React ignores your input. This is intentional. By writing `<input value={filterText} />`, you've set the `value` prop of the `input` to always be equal to the `filterText` state passed in from `FilterableProductTable`. Since `filterText` state is never set, the input never changes.

You want to make it so whenever the user changes the form inputs, the state updates to reflect those changes. The state is owned by `FilterableProductTable`, so only it can call `setFilterText` and `setInStockOnly`. To let `SearchBar` update the `FilterableProductTable`'s state, you need to pass these functions down to `SearchBar`:

```js {2,3,10,11}
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

Inside the `SearchBar`, you will add the `onChange` event handlers and set the parent state from them:

```js {5}
<input 
  type="text" 
  value={filterText} 
  placeholder="Search..." 
  onChange={(e) => onFilterTextChange(e.target.value)} />
```

Now the application fully works!

<Sandpack>

```jsx App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Search..." 
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding: 4px;
}
td {
  padding: 2px;
}
```

</Sandpack>

You can learn all about handling events and updating state in the [Adding Interactivity](/learn/adding-interactivity) section.

## Where to go from here {/*where-to-go-from-here*/}

This was a very brief introduction to how to think about building components and applications with React. You can [start a React project](/learn/installation) right now or [dive deeper on all the syntax](/learn/describing-the-ui) used in this tutorial.

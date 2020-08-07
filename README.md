# tsgraph

Below is a rough idea as to goal for functionality. 

Please treat all 0.x.y revisions to be unstable

## Define Graph values

We can define partial named nodes by specifying a scheme with a default amount of imformation
Each scheme will have its own rules on when a name is valid, so we only need to know the scheme, if the scheme can be any, meaning 
it could also be a blank node, then the value should start with `:` the value can be named as `:such`

### Nodes

```tsgraph
export const partialNode = :
export const blankNode = :named
export const scheme = scheme:
export const httpsPartial = https:
export const httpsTemplate = https://example.com/path/
```

```tsgraph
import * as ns from "./nodes/example1"

console.log({ ns })
```

```bash
node n0.tsgraph
```

### Literals

Literals can be defined by using a value of that type, or their primative name or `Date`, literal definitions must be 
defined within a graph, which is expanded more on later

```
export const boolean = graph { boolean }
export const booleanTrue = graph { true }
export const booleanFalse = graph { false }
export const number = graph { number }
export const numberZero = graph { 0 }
export const numberOne = graph { 0 }
export const string = graph { string }
export const stringEmpty = graph { "" }
export const stringWhatever = graph { "Whatever" }
export const stringWhatever = graph { "Whatever" }
export const bigint = graph { bigint }
export const date = graph { Date }
```

```tsgraph
import * as from "./literals/example1"

console.log({ ns })
```

```bash
node n0.tsgraph
```

### References

Named nodes can be extended using this pattern

```tsgraph
const domain = https://example.com

const path = {domain}/path
```

### Context

Every guide must have a context referenced before any other dependent graph is defined, as it is referenced
(it isn't a hard requirement, but helps out), the `guide:?domain` parameter defines the domain as such

Define module cpmyexy, this is the default graph once defined can be referenced as normal

`guide:node?required&domain`
```tsgraph
export const context = https://
```

Context can be referenced when defining other named nodes

```tsgraph
const author = {context}/author
```

By default `context` is refered to if only a path is defined

The above could be defined as 

```tsgraph
const author = /author
```

#### New context

If a different context is to be used, a local `context` variable can be defined

```tsgraph
const context = https://different.site

const differentSiteAuthor = /author

console.log({ differentSiteAuthor })
console.log({ withinContext: differentSiteAuthor.value.startsWith(context.value) })
```

### Graph

An empty graph can be defined as

```tsgraph
graph { }
```

By default the subject of the graph is a blank node, the subject of the graph can be defined as an argument using `as`

```tsgraph
graph as subject {

}
```

Required varaibles could be defined within the graph parameters

```tsgraph
graph (variableA: string, variableB: number, name: NamedNode) {

}
```

These variables come after the subject for the graph

```tsgraph
graph as subject (variable: string) {

}
```

Variables can be defaulted

```tsgraph
graph as subject (variable: string, target: NamedNode = subject) {

}
```

A graph can be a single value 

```tsgraph
graph {
	object
}
```

If a value is an expression, it should be surrounded by `()`

```tsgraph
graph {
	(a + b)
}
```

Predicates that relate to the current context can be defined as key value pairs

```tsgraph
graph {
	predicate object
}
```

Triples can be defined as `subject predicate object`

```tsgraph
graph {
	predicate object
}
```

Sub graphs can be defined using

```tsgraph
graph {
	predicate as subject {
		predicate object
	}
}
```

#### Extending graphs

A graph can extend another graph and overwrite a predicate or value within 

```tsgraph
graph as stringGraph {
	""
}

graph as nameStringGraph extends stringGraph {
	"Named!"
}

graph as templateGraph {
	name ""
}

graph as namedGraph extends templateGraph {
	name "Named!"
}

graph as namedWithAnotherGraph extends templateGraph {
	name nameStringGraph
}
```

A graph can be re-defined without providing an implementation

```tsgraph
graph as templateGraphNew extends templateGraph {
	
}

graph namedGraphNew extends templateGraphNew {
	name: "New Name!"
}
```

A graph can extend multiple graphs 

```tsgraph
graph a {

}

graph b {

}

graph c extends a, b {

}
```

Variables still come directly after the subject definition or `graph`

```tsgraph
graph as namedGraph (variable: string) extends templateGraph {
	name variable
}
```

All variables from the template graph will be available within the extended graph

```tsgraph
graph as anotherGraph extends namedGraph {
	another variable
} 
```

Additional variables can also be defined

```tsgraph
graph as yetAnotherGraph(newVariable: string) extends anotherGraph {
   yetAnother newVariable
}
```

#### Graph function

A graph function allows for effects to be applied to the graph based on changes to the graph, within these graphs, changes may be queried
and additional changes can be made

Instead of using `as`, you can define a function using `graph`, parameters and `extends` operate the same

A function graph can reference itself using `this`

```tsgraph
graph function f {
	console.log([ ...this ])

	yield { name "Hello!" }

	console.log([ ...this ])
}
```

Graph subjects can be defined inline

```tsgraph
graph function /message {
	yield { "Hello World" }
}
```

Which can then be directly referenced in new graphs

```tsgraph
graph {
	message /message
}
```

New values for the graph are provided using `yield`, which can be used in place of `graph`, for example `yield function {}` is valid syntax

```tsgraph
graph as named {
  name string
}

export graph function f(nameValue: string) extends named {
	yield {
		name nameValue
	}
}
```

Graph functions that extend other functions can reference the extending graph using `super`

```tsgraph
import { f }

graph function fNew extends f {
	console.log([ ...this ])

	// This matches fNew with f, this can provide a default for all values defined
	yield super

	console.log([ ...this ])

	// This updates the values
	yield { name "New!" }

	console.log([ ...this ])
}
```

A function graph can contain any value syntax that a normal function may contain, including `await`

```tsgraph
export graph status {
	status string
}

graph function f extends status {

	yield {
		status "Loading"
	}

	const response = await fetch("https://httpstatuses.com/200")

	yield {
		status (response.ok ? "OK" : "Not OK")
	}

}
```

Functions can be modified as to how they operate, `catch` and `finally` can be used to clean up previous changes

_below is a maybe_

```tsgraph
import { status }

graph function f extends status {
	try {
		yield { status "OK" }
	} catch (e) {
		yield { status "Caught error" }
	} finally {
		yield { status "Done" }
	}
}
```

# Observing

## Lens

A lens is like a function, but instead of just `yield` the `lens` can use `return`

```
lens as subject {
  return 1
}

get subject
```

By default the graph defined is related to `context`

Yet, `this` can be used a special parameter to define the graph type

```
export graph named {
	name string
}

export lens as scalar(this: named, predicate: NamedNode | BlankNode) {
	for (const { object } of this.match({ predicate })) {
		return object
	}
	return undefined
}

lens as a {
	return 1
}

export lens as b {
	return 1 + get a
}
```

`is` scan be used to define a lens that is the same as another lens, with a default input graph

```tsgraph
import { scalar } 

export lens as name(this: named) is scalar {
	predicate name
}
```

Lenses `this` is passed down unless specified

```tsgraph
import { name } 

export lens as c(this: named)  {
	return get b + get name
}
```

```tsgraph
import { c }

graph as e {
	variable 2
}

lens as d(this: named) {
	return [
		`User ${get c with e}`,
		`User ${get c { variable 2 } }`
	].join(", ")
}
```

A lens can be referenced within a graph

```tsgraph
import { named }

graph as module extends named {
	name string
	version string
}

graph as moduleInstance extends module {
	name "Module"
	version "1.0.0"
}

lens as version(this: module) is scalar {
	predicate version
}

// This binds `application` to the current `context` graph
graph as application {
    containedModule as moduleInstance {
		(`${get name}:${get version}`)
	}
}



```

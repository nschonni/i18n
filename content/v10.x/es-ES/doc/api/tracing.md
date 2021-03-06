# Seguimiento de eventos

<!--introduced_in=v7.7.0-->

> Estabilidad: 1 - Experimental

Trace Event provides a mechanism to centralize tracing information generated by V8, Node.js core, and userspace code.

Tracing can be enabled with the `--trace-event-categories` command-line flag or by using the `trace_events` module. The `--trace-event-categories` flag accepts a list of comma-separated category names.

Las categorías disponibles son:

* `node` - Un marcador de posición vacío.
* `node.async_hooks` - Permite la captura de rastros de datos detallados [`async_hooks`]. The [`async_hooks`] events have a unique `asyncId` and a special `triggerId` `triggerAsyncId` property.
* `node.bootstrap` - Permite la captura de los hitos bootstrap de Node.js.
* `node.fs.sync` - Permite la captura de datos de seguimientos para métodos de sincronización de archivos del sistema.
* `node.perf` Permite la captura de las mediciones de [Desempeño del API](perf_hooks.html). 
  * `node.perf.usertiming` - Enables capture of only Performance API User Timing measures and marks.
  * `node.perf.timerify` - Enables capture of only Performance API timerify measurements.
* `node.promises.rejections` - Enables capture of trace data tracking the number of unhandled Promise rejections and handled-after-rejections.
* `node.vm.script` - Enables capture of trace data for the `vm` module's `runInNewContext()`, `runInContext()`, and `runInThisContext()` methods.
* `v8` - The [V8](v8.html) son eventos recolectores de basura, compilan, y están relacionados con la ejecución.

Por defecto están activas las categorías `node`, `node.async_hooks`, y `v8`.

```txt
node --trace-event-categories v8,node,node.async_hooks server.js
```

Prior versions of Node.js required the use of the `--trace-events-enabled` flag to enable trace events. Este requisito ha sido removido. However, the `--trace-events-enabled` flag *may* still be used and will enable the `node`, `node.async_hooks`, and `v8` trace event categories by default.

```txt
node --trace-events-enabled

// es equivalente a

node --trace-event-categories v8,node,node.async_hooks
```

Alternamente, los eventos de seguimiento pueden ser permitidos utilizando el módulo `trace_events`:

```js
const trace_events = require('trace_events');
const tracing = trace_events.createTracing({ categories: ['node.perf'] });
tracing.enable();  // Activa la captura de eventos de seguimiento para la categoría 'node.perf'

// realiza trabajo

tracing.disable();  // Desactiva la captura de eventos para la categoría 'node.perf'
```

Running Node.js with tracing enabled will produce log files that can be opened in the [`chrome://tracing`](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool) tab of Chrome.

The logging file is by default called `node_trace.${rotation}.log`, where `${rotation}` is an incrementing log-rotation id. The filepath pattern can be specified with `--trace-event-file-pattern` that accepts a template string that supports `${rotation}` and `${pid}`:

```txt
node --trace-event-categories v8 --trace-event-file-pattern '${pid}-${rotation}.log' server.js
```

Starting with Node.js 10.0.0, the tracing system uses the same time source as the one used by `process.hrtime()` however the trace-event timestamps are expressed in microseconds, unlike `process.hrtime()` which returns nanoseconds.

## El módulo `trace_events`

<!-- YAML
added: v10.0.0
-->

### Objeto de `seguimiento`

<!-- YAML
added: v10.0.0
-->

The `Tracing` object is used to enable or disable tracing for sets of categories. Instances are created using the `trace_events.createTracing()` method.

Cuando es creado, el objeto de `seguimiento` está deshabilitado. Calling the `tracing.enable()` method adds the categories to the set of enabled trace event categories. Calling `tracing.disable()` will remove the categories from the set of enabled trace event categories.

#### `tracing.categories`

<!-- YAML
added: v10.0.0
-->

* {string}

A comma-separated list of the trace event categories covered by this `Tracing` object.

#### `tracing.disable()`

<!-- YAML
added: v10.0.0
-->

Deshabilita el objeto `Tracing`.

Only trace event categories *not* covered by other enabled `Tracing` objects and *not* specified by the `--trace-event-categories` flag will be disabled.

```js
const trace_events = require('trace_events');
const t1 = trace_events.createTracing({ categories: ['node', 'v8'] });
const t2 = trace_events.createTracing({ categories: ['node.perf', 'node'] });
t1.enable();
t2.enable();

// Imprime 'node,node.perf,v8'
console.log(trace_events.getEnabledCategories());

t2.disable(); // Solo deshabilita la emisión de eventos para la categoría 'node.perf'

// Imprime 'node,v8'
console.log(trace_events.getEnabledCategories());
```

#### `tracing.enable()`

<!-- YAML
added: v10.0.0
-->

Enables this `Tracing` object for the set of categories covered by the `Tracing` object.

#### `tracing.enabled`

<!-- YAML
added: v10.0.0
-->

* {boolean} `true` solo si el objeto `Tracing` ha sido habilitado.

### `trace_events.createTracing(options)`

<!-- YAML
added: v10.0.0
-->

* `options` {Object} 
  * `categories` {string[]} Un conjunto de nombres de categorías de seguimiento. Values included in the array are coerced to a string when possible. An error will be thrown if the value cannot be coerced.
* Devuelve: {Tracing}

Crea y devuelve un objeto `Tracing` para el conjunto de `categories` dado.

```js
const trace_events = require('trace_events');
const categories = ['node.perf', 'node.async_hooks'];
const tracing = trace_events.createTracing({ categories });
tracing.enable();
// do stuff
tracing.disable();
```

### `trace_events.getEnabledCategories()`

<!-- YAML
added: v10.0.0
-->

* Devuelve: {string}

Returns a comma-separated list of all currently-enabled trace event categories. The current set of enabled trace event categories is determined by the *union* of all currently-enabled `Tracing` objects and any categories enabled using the `--trace-event-categories` flag.

Given the file `test.js` below, the command `node --trace-event-categories node.perf test.js` will print `'node.async_hooks,node.perf'` to the console.

```js
const trace_events = require('trace_events');
const t1 = trace_events.createTracing({ categories: ['node.async_hooks'] });
const t2 = trace_events.createTracing({ categories: ['node.perf'] });
const t3 = trace_events.createTracing({ categories: ['v8'] });

t1.enable();
t2.enable();

console.log(trace_events.getEnabledCategories());
```
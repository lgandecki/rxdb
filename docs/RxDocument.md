# RxDocument
A document is a single object which is stored in a collection. It can be compared to a single record in a relational database table.


## insert
To insert a document into a collection, you have to call the collection's .insert()-function.
```js
myCollection.insert({
  name: 'foo',
  lastname: 'bar'
});
```

## find
To find document in a collection, you have to call the collection's .find()-function.
```js
myCollection.find().exec() // <- find all documents
  .then(documents => console.dir(documents));
```


## Functions

### get()
This will get a single field of the document. If the field is encrypted, it will be automatically decrypted before returning.

```js
var name = myDocument.get('name'); // returns the name
```

### proxy-get
As RxDocument is wrapped into a [Proxy-object](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Proxy), you can also directly access values instead of using the get()-function.

```js
  var name = myDocument.name;

  var nestedValue = myDocument.whatever.nestedfield;
```

### set()
To change data in your document, use this function. It takes the field-path and the new value as parameter. Note that calling the set-function will not change anything in your storage directly. You have to call .save() after to submit changes.

```js
myDocument.set('firstName', 'foobar');
console.log(myDocument.get('firstName')); // <- is 'foobar'
```

### proxy-set
As RxDocument is wrapped into a [Proxy-object](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Proxy), you can also directly set values instead of using the set()-function.

```js
myDocument.firstName = 'foobar';
myDocument.whatever.nestedfield = 'foobar2';
```

### save()
This will store the document in the storage if it has been changed before. Call this every time after calling the set-method.
```js
myDocument.name = 'foobar';
await myDocument.save(); // submit the changes to the storage
```

### remove()
This removes the document from the collection.
```js
myDocument.remove();
```

### Observe $
Calling this will return an [rxjs-Observable](http://reactivex.io/rxjs/manual/overview.html#observable) which emits all change-Events belonging to this document.

```js
// get all changeEvents
myDocument.$()
  .subscribe(changeEvent => console.dir(changeEvent));
```

### get$()
This function returns an observable of the given paths-value.
The current value of this path will be emitted each time the document changes.
```js
// get the life-updating value of 'name'
var isName;
myDocument.get$('name')
  .subscribe(newName => {
    isName = newName;
  });

myDocument.set('name', 'foobar2');
await myDocument.save();

console.dir(isName); // isName is now 'foobar2'
```

### proxy-get$
You can directly get value-observables for a fieldName by adding the dollarSign ($) to its name.

```js
// top-level
var currentName;
myDocument.firstName$
  .subscribe(newName => {
    currentName = newName;
  });
myDocument.firstName = 'foobar2';
await myDocument.save();
console.dir(currentName); // currentName is now 'foobar2'

// nested
var currentNestedValue;
myDocument.whatever.nestedfield$
  .subscribe(newName => {
    currentNestedValue = newName;
  });
myDocument.whatever.nestedfield = 'foobar2';
await myDocument.save();
console.dir(currentNestedValue); // currentNestedValue is now 'foobar2'
```

### deleted$
Emits a boolean value, depending on whether the RxDocument is deleted or not.

```js
let lastState = null;
myDocument.deleted$.subscribe(state => lastState = state);

console.log(lastState);
// false

await myDocument.remove();

console.log(lastState);
// true
```

### get deleted
A getter to get the current value of `deleted$`.

```js
console.log(myDocument.deleted);
// false

await myDocument.remove();

console.log(myDocument.deleted);
// true
```


### synced$
Emits a boolean value of whether the RxDocument is in the same state as its value stored in the database.
This is useful to show warnings when two or more users edit a document at the same time.

Browser tab A
```js
let lastState = null;
myDocument.synced$.subscribe(state => lastState = state);
console.log(lastState);
// true

myDocument.firstName = 'foobar';
console.log(lastState);
// true
```

Browser tab B
```js
myDocument.firstName = 'barfoo';
await myDocument.save();
```

Browser tab A
```js
console.log(lastState);
// false
```

<details>
<summary>
  <b>Example with Angular 2</b>
</summary>

```html
<div *ngIf="!(hero.synced$ | async)">
    <h4>Warning:</h4>
    <p>Someone else has <b>changed</b> this document. If you click save, you will overwrite the changes.</p>
    <button md-raised-button color="primary" (click)=hero.resync()>resync</button>
</div>
```

![synced.gif](files/synced.gif)
</details>

### get synced
A getter to get the current value of `synced$`.

Browser tab A
```js
console.log(myDocument.synced);
// true

myDocument.firstName = 'foobar';
console.log(myDocument.synced);
// true
```

Browser tab B
```js
myDocument.firstName = 'barfoo';
await myDocument.save();
```

Browser tab A
```js
console.log(myDocument.synced);
// false
```

### resync()
If the RxDocument is not in sync (synced$ fires `false`), you can run `resync()` to overwrite own changes with the new state from the database.

```js
myDocument.firstName = 'foobar';

// now someone else overwrites firstName with 'Alice'

myDocument.resync();

console.log(myDocument.firstName);
// Alice
```

---------
If you are new to RxDB, you should continue [here](./RxQuery.md)

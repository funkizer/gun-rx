# Gun-RX
Learning Gun DB while wrapping it in RXJS Observables & Typescript.

[![Join the chat at https://gitter.im/gun-angular/Lobby](https://badges.gitter.im/gun-angular/Lobby.svg)](https://gitter.im/gun-angular/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## What's this?

- Very early, don't use in production. Feel free to look, play around and let me know what you think.
- Aims to wrap all gun methods in [RXJS](https://github.com/reactivex/rxjs) Observables
- Should work with Angular, React, any framework, or without any. 
- "Pet project" developed on limited free time.
- Like Gun, this runs fine entirely in a browser, without any server. To sync and persist data, you need at least one. Follow instructions on [github.com/amark/gun](https://github.com/amark/gun) on how to spin up a NodeJS server.

## What's Gun?
A realtime p2p database built on JS that runs in a browser and server(s), created by [Mark Nadal / @amark](https://github.com/amark). Graphs, documents, key-value, you name it. Read more: [gun.js.org](http://gun.js.org)

## Links
- [Gun](http://gun.js.org)
- [Gitter](https://gitter.im/gun-rx)
- [GitHub](https://github.com/funkizer/gun-rx)
- [Twitter: @funkizer](https://twitter.com/funkizer)
- [Gun Gitter](https://gitter.im/amark/gun)
- [Gun GitHub](https://github.com/amark/gun)
- [Learn more @ Youtube](https://www.youtube.com/results?search_query=mark+nadal)

## Using without a framework
I'm mainly using Angular to dev & test this. Let me know what happens if you try it without NG!

### TypeScript / Vanilla JS
The lib is built to ES6 JS, so use your favourite build/transpile tool to support other than latest evergreen browsers.

```
import { GunRef } from 'gun-rx';
const gunRef = new GunRef();
// This is your "root" GunRef, you should probably keep it safe and reuse
```
To pass options (optional):
```
import { GunRef, GunOptions } from 'gun-rx';
const gunOptions: GunOptions = {
  peers: [location.origin + '/gun']
};
const gunRef = new GunRef(gunOptions);
// Will call gun.opt(gunOptions) for you
```

```
const subscription = this.db.get('test')
  .on()
  .subscribe(data => { doSomethingWith(data) });
this.db.get('test').put({testing: true});
```

.on() returns a hot Observable which keeps streaming changes to you indefinately. So remember to call `subscription.unsubscribe()` when the view is destroyed or you don't want any more updates. Not doing this will result in memory leaks and other unwanted stuff to happen.

## Using with NodeJS?
I don't see why this wouldn't *just work* on the server, as long as your NodeJS version is recent enough to support import/export. Let me know if you try.

## Using with Angular
If you don't have an angular project yet, run `ng new my-gun-rx-app`

CD into your project root folder, run `npm install gun-rx --save`

Add into your (App)Module:

```
import { GunAngularModule, GunOptions } from 'gun-rx';
const gunOptions: GunOptions = {
  peers: [location.origin + '/gun']
};
@NgModule({
  // ...
  imports: [
   GunAngularModule.forRoot(gunOptions),
    // ...
  ],
```

Inject GunRef to a component:

```  
@Component() export class MyComponent implements OnInit, OnDestroy {
  private subs:Subscription[] = [];
  constructor(private db: GunRef) { } 
  onInit() {
    const testNode = this.db.get('test');
    this.subs = [
      // Keeps pushing values upon changes to this node:
      testNode.on().subscribe(data => { doSomethingWith(data) })
    ];

    // Gets data once and completes:
    testNode.val().subscribe(data => { doSomethingWith(data) });
    
    testNode.put({testing: true});
    setTimeout(()=> {
      testNode.put({testing: true, timestamp: new Date().getTime()})
    }, 2000);
  }   
```
**Always** unsubscribe when you leave the component, to avoid memory leaks:
```
  onDestroy() {
    this.subs.forEach(sub => sub.unsubscribe());
  }
```

**Better yet:** Use the awesome Angular's async pipe with the `*ngIf; let` -syntax. Then you don't need to worry about subscribing & unsubscribing yourself.
```
   data$: Observable<any>
   onInit() {
     this.data$ = this.db.get('test').on();
   }
```
And in your template:
```
<div *ngIf="(data$|async); let data">
  Now *data* is a plain JS object or array in this div's scope:
  {{ data|json }}
</div>
```

## Running with this repo
- `git clone https://github.com/funkizer/gun-rx`
- `cd gun-rx`
- `ng serve` 
- With AOT compilation: `npm run serve-aot`

## Targets

- Provide an API as close as possible to Gun, with strong typing, Observables and documenting-by-code (intentful names, good comments)
- Preserve Gun's logic in chaining method calls
- Evolve as Gun evolves
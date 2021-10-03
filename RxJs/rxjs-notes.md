# RxJs 

## Tips

1. Una intro fundamental a reactive programming: https://gist.github.com/staltz/868e7e9bc2a7b8c1f754
2. Usa siempre al final del último operador un catchError, asi se evita que el error se propague por toda la secuencia
3. De la misma manera, no basta con poner un catchError global, pues si por ejemplo tenemos un mergeMap, deberíamos por uno al final del propio inner observable, sobretodo para que el error no se propague y rompa al outer observable

Ejemplo:

```js
interval(1000).pipe(
  mergeMap((i) => of(i)),
  finalize(() => 'Errored or completed')
).subscribe(console.log)
```

Si `of(1)` da error, el flujo enero se rompe.

Se puede evitar de la siguiente manera:

```js
interval(1000).pipe(
  mergeMap((i) => defer(() => {
    if (i === 2) throwError('Error');
    return of(2);
  }).pipe(
    catchError(() => of(2)),
  )),
  finalize(() => 'Errored or completed')
).subscribe(console.log)
```

3. Add "takeUntil" as the last operator: de esta manera nos aseguramos que siempre una secuencia se podría completar.
   1. Si tenemos un mergeMap, si el outer observable se completa, eso no quiere decir que lo haga el inner.
4. Don't unsubscribe too much: https://benlesh.medium.com/rxjs-dont-unsubscribe-6753ed4fda87
5. `vm$`pattern: se trata de, en vez de tener en tu clase 3 variables tipo (page$, limit$, currentPage$), tengas un:

```js
// JS
vm$ = combineLatest([poge$, limit$, currentPage$]).pipe(
  map(([page, lmit, currentPage]) => ({ page, limit, currentPage }))
);
```

```html
// HTML
<ng-container ngIf="vm$ | async as vm">
  <div>{{ vm.page }}</div>
</ng-container>
```

Y en la vista, en vez de tener tres `async` pipes, una para cada observable, solo tenemos una para el `$vm`y lo ponemos en el 
tag html padre.

6. Si haces un observable service (https://blog.angular-university.io/how-to-build-angular2-apps-using-rxjs-observable-data-services-pitfalls-to-avoid/)
   1. Si trabajas con subjects no expongas el subject entero sino `.asObservable()`
   2. En ese caso, todos los `.next()` deberían realizarse dentro del servicio, no en el componente que lo implemente.
7. Recordar que los Subject son tambien Observers (next, complete, error), por lo que se pueden usar en `tap` o `subscription`directamente
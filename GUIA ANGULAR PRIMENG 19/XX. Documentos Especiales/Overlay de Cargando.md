1. **Servicio de carga (singleton)**
2. 
```ts
// src/app/core/services/loading.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class LoadingService {
  private count = 0;
  private loadingSubject = new BehaviorSubject<boolean>(false);
  loading$ = this.loadingSubject.asObservable();

  show(): void {
    this.count++;
    if (this.count === 1) this.loadingSubject.next(true);
  }

  hide(): void {
    if (this.count === 0) return;
    this.count--;
    if (this.count === 0) this.loadingSubject.next(false);
  }
}
```

2. **Interceptor HTTP**

```ts
// src/app/core/interceptors/loading.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { LoadingService } from '../services/loading.service';
import { inject } from '@angular/core';
import { delay, finalize } from 'rxjs';

export const LoadingInterceptor: HttpInterceptorFn = (req, next) => {
	const loadingService: LoadingService = inject(LoadingService);
	loadingService.show();
	return next(req).pipe(
		catchError((error) => {
			return throwError(() => error);
		}),
		finalize(() => {
			loadingService.hide();
		})
	);
};
```

3. **Componente de overlay**


> [!NOTE] Nota
> Este componente se puede dividir en archivo de plantilla `loading.component.html` y estilos `loading.component.scss` 


```ts
// src/app/shared/components/loading.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProgressSpinnerModule } from 'primeng/progressspinner';
import { LoadingService } from '../core/loading.service';
import { Subscription } from 'rxjs';

@Component({
  selector: 'app-loading',
  standalone: true,
  imports: [CommonModule, ProgressSpinnerModule],
  style: `
	.loading-overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.5);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 9999;
	}

	.loading-content {
		display: flex;
		align-items: center;
		gap: 1rem;
		color: white;
		font-size: 1.2rem;
	}
`,
  template: `
    <div *ngIf="loading"
         class="fixed inset-0 bg-black/50 loading-overlay flex items-center flex-wrap justify-center z-50">
      <p-progressSpinner></p-progressSpinner>
      <span class="ml-4 text-white text-xl">Cargando...</span>
    </div>
  `,
})
export class LoadingComponent implements OnInit, OnDestroy {
	loading = false;
	private loadingSubscription!: Subscription;
	
	constructor(private loadingService: LoadingService) {}
	
	ngOnInit() {
		this.loadingSubscription = this.loadingService.loading$.subscribe(
		(val) => (this.loading = val)
		);
	}
	
	ngOnDestroy() {
		if (this.loadingSubscription) {
			this.loadingSubscription.unsubscribe();
		}
	}
}
```

4. **Registro de interceptor y bootstrap standalone**
En los providers de `app.config.ts` agregar el interceptor de `LoadingInterceptor` 
```ts
// src/app/app.config.ts

  providers: [
    provideHttpClient(withInterceptors([LoadingInterceptor])),
  ]
```

5. **Inclusión del componente en AppComponent**
En `app.component` (ya sea en la plantilla `html` o el atributo  `template`) agregar el `<app-loading> 
```ts
// src/app/app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { LoadingComponent } from './shared/loading.component';

@Component({
  standalone: true,
  imports: [RouterOutlet, LoadingComponent],
  template: `
    <router-outlet></router-outlet>
    <app-loading></app-loading>
  `,
})
export class AppComponent {}
```

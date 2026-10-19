# Reference — Angular

> Dibaca oleh skill bila `stack-profile.md` → `stack: angular`. Susunan section sama dengan `react.md` dan `nextjs.md`.
> **Status: tervalidasi** dengan template `fe-starter-angular` (Angular 21.2, standalone, zoneless, signals, Vitest): lint + angular-eslint + boundaries, typecheck, 44 test, build production (HTTP) & development (mock) lolos. Contoh kode di bawah diambil dari template itu.

## 1. Metode yang direkomendasikan

| Aspek | Pilihan | Alasan |
|---|---|---|
| Versi | **Angular 21 (LTS)**; Angular 22 butuh Node `^22.22.3`/`^24.15.0` | 21 jalan di Node `^20.19`, `^22.12`, `>=24` |
| Komponen | **Standalone**, `ChangeDetectionStrategy.OnPush`, **zoneless**, signals, control flow `@if/@for/@switch` | Default `ng new` modern; tanpa NgModule dan zone.js |
| DI | **DI bawaan Angular**; port = `abstract class` (langsung jadi token) | Tidak perlu container sendiri |
| Registrasi | **`providers` di route fitur** (`<feature>.routes.ts`), lazy lewat `loadChildren` | Fitur terisolasi; adapter tidak global |
| Mock vs HTTP | **Diganti saat build** lewat `fileReplacements` (`<feature>.adapters.ts` → `.adapters.mock.ts`) | Kode mock tidak ikut bundle production (flag runtime tidak di-tree-shake — terverifikasi) |
| State | **Signal store per fitur** (`@Injectable` + `signal`/`computed`) | Ringan; NgRx SignalStore bila tim butuh struktur/devtools |
| Kontrak data | `Promise<Result<T, AppError>>` (adapter memakai `firstValueFrom`) | Portable lintas stack; `Observable` hanya untuk stream (§ 14) |
| HTTP | `provideHttpClient(withFetch(), withInterceptors([...]))`; interceptor base URL/auth di `core/http` | Satu tempat URL & token |
| Form | **Typed Reactive Forms** | Validasi di boundary |
| Atom UI | **Attribute directive** pada elemen native (`<button appButton>`) + komponen kecil (`app-card`) | Semantik & a11y native tetap; styling satu sumber |
| Styling | Tailwind CSS 4 via `@tailwindcss/postcss` (`.postcssrc.json`) | Sama dengan stack lain |
| Test | **`ng test`** (Vitest bawaan Angular 21, globals) + `TestBed` + `HttpTestingController` | Tanpa dependency test tambahan |

**Skill vendor (opsional)** — `web-design-guidelines` (Vercel) bersifat framework-agnostic dan bisa dipakai untuk review UI/a11y bersama `bp-ui-components`. Untuk praktik khusus Angular belum ada skill vendor yang direkomendasikan; acuan utama dokumentasi resmi (angular.dev) dan Angular CLI MCP server (`ng new` membuat `.vscode/mcp.json`). Bila bertentangan dengan skill `bp-*`, skill `bp-*` yang berlaku.

## 2. Folder map

```
src/
├── main.ts                   # bootstrapApplication — lint `app`
├── app/                      # lint `app`: app.config.ts, app.routes.ts, home-page.ts
├── core/
│   ├── config/               # environment.ts (production) / environment.development.ts (fileReplacements)
│   └── http/                 # api-endpoints.ts, to-app-error.ts (+ Envelope), api-base-url.interceptor.ts
├── shared/kernel/            # result.ts, app-error.ts — TS murni
├── shared/lib/               # format.ts (formatOrDash), error-messages.ts (toUiError) — TS murni
├── shared/ui/                # atom (directive/komponen) + or-dash.pipe.ts
└── features/<feature>/
    ├── domain/               # types + pure functions (no @angular/*, no rxjs)
    ├── application/          # ports (abstract class) + use cases (only when logic exists)
    ├── infrastructure/       # adapters HTTP & mock, mapper, constants, <feature>.adapters(.mock).ts
    ├── presentation/         # <feature>.store.ts + pages/components
    └── <feature>.routes.ts   # lint `app`: route + providers (the only feature file that names adapters)
```

Angular tidak memakai elemen lint `di` (komponen/store memakai `inject()`); folder `core/di` tidak dibuat. Pipe `orDash` ada di `shared/ui`, bukan `shared/lib`, karena `shared/lib` dilarang meng-import `@angular/*`.

## 3. Kernel

`Result`, `ok`, `err`, `AppError` sama dengan `react.md` § 3, **tanpa** `AppErrorException`/`unwrap` (store membaca `Result` langsung). `shared/lib/format.ts` (`formatOrDash`) dan `shared/lib/error-messages.ts` (`toUiError`) identik dengan stack lain; template memakainya lewat pipe:

```ts
// shared/ui/or-dash.pipe.ts
@Pipe({ name: 'orDash' })
export class OrDashPipe implements PipeTransform {
  transform<T>(value: T | null | undefined, format?: (value: T) => string): string {
    return formatOrDash(value, format);
  }
}
// template: {{ invoice.amount | orDash: formatCurrency }} — component exposes `protected readonly formatCurrency = formatCurrency`
```

## 4. Domain

Sama persis dengan `nextjs.md` § 4 / `react.md` § 4 (`Bank`, `selectableBanks`, `MIN_TOPUP`, `validateAmount`) — file domain berpindah antar stack tanpa perubahan.

## 5. Port

```ts
// features/topup/application/bank.repository.ts — abstract class = DI token
export abstract class BankRepository {
  abstract getBanks(): Promise<Result<Bank[], AppError>>;
}
```

## 6. Adapter + mapper

Mapper (`toBank`, `toBanks`) dan konstanta kode bisnis sama dengan stack lain. Adapter:

```ts
// features/topup/infrastructure/http-bank.repository.ts
@Injectable()
export class HttpBankRepository extends BankRepository {
  private readonly http = inject(HttpClient);

  async getBanks(): Promise<Result<Bank[], AppError>> {
    let body: Envelope<BankApiItem[]>;
    try {
      body = await firstValueFrom(this.http.get<Envelope<BankApiItem[]>>(API.topup.banks));
    } catch (error) {
      return err(toAppError(toHttpFailure(error), TOPUP_ERROR_TYPES));   // HTTP / network failure
    }
    if (body?.status?.code !== TOPUP_SUCCESS.OK) return err(toAppError({ status: 200, code: body?.status?.code }, TOPUP_ERROR_TYPES));
    const banks = toBanks(body.data ?? []);
    return banks ? ok(banks) : err({ type: 'server', message: 'Invalid bank data from server' });
  }
}
```

```ts
// core/http/to-app-error.ts — HttpErrorResponse → transport failure (status 0 = network/CORS)
export const toHttpFailure = (error: unknown): HttpFailure => {
  if (!(error instanceof HttpErrorResponse) || error.status === 0) return { network: true };
  const body = error.error as Envelope<unknown> | null;
  return { status: error.status, code: body?.status?.code };
};
// toAppError(failure, businessCodes): identical to nextjs.md § 6
```

Mock adapter (`MockBankRepository extends BankRepository`) memakai data mentah berbentuk sama dan **mapper yang sama**.

## 7. Use case (hanya bila ada logika)

```ts
// features/topup/application/create-topup.usecase.ts — plain class, no decorators
export class CreateTopupUseCase {
  constructor(private readonly banks: BankRepository, private readonly topups: TopupRepository) {}

  async execute(input: CreateTopupInput): Promise<Result<TopupInvoice, AppError>> {
    const amountError = validateAmount(input.amount);
    if (amountError) return err(amountError);
    const banks = await this.banks.getBanks();
    if (!banks.ok) return banks;
    if (!selectableBanks(banks.value).some((bank) => bank.code === input.bankCode)) {
      return err({ type: 'validation', message: 'Selected bank is not available' });
    }
    return this.topups.create(input);
  }
}
```

## 8. DI / composition root

```ts
// features/topup/infrastructure/topup.adapters.ts — production (HTTP)
export { HttpBankRepository as BankAdapter } from './http-bank.repository';
export { HttpTopupRepository as TopupAdapter } from './http-topup.repository';

// features/topup/infrastructure/topup.adapters.mock.ts — swapped in by the development configuration
export { MockBankRepository as BankAdapter } from './mock-bank.repository';
export { MockTopupRepository as TopupAdapter } from './mock-topup.repository';
```

```json
// angular.json → projects.<app>.architect.build.configurations.development
"fileReplacements": [
  { "replace": "src/core/config/environment.ts", "with": "src/core/config/environment.development.ts" },
  { "replace": "src/features/topup/infrastructure/topup.adapters.ts", "with": "src/features/topup/infrastructure/topup.adapters.mock.ts" }
]
```

```ts
// features/topup/topup.routes.ts — lint element `app`; nothing imports it except app.routes.ts
export const TOPUP_ROUTES: Routes = [
  {
    path: '',
    providers: [
      { provide: BankRepository, useClass: BankAdapter },
      { provide: TopupRepository, useClass: TopupAdapter },
      {
        provide: CreateTopupUseCase,
        useFactory: (banks: BankRepository, topups: TopupRepository) => new CreateTopupUseCase(banks, topups),
        deps: [BankRepository, TopupRepository],
      },
      TopupStore,
    ],
    loadComponent: () => import('./presentation/topup-page').then((m) => m.TopupPage),
  },
];

// app/app.config.ts
providers: [provideBrowserGlobalErrorListeners(), provideHttpClient(withFetch(), withInterceptors([apiBaseUrlInterceptor])), provideRouter(routes, withComponentInputBinding())]
```

Setiap fitur baru: buat `<feature>.adapters.ts` + `.adapters.mock.ts` dan tambahkan entri `fileReplacements`.

## 9. State / store

```ts
// features/topup/presentation/topup.store.ts — provided in topup.routes.ts, never providedIn 'root'
@Injectable()
export class TopupStore {
  private readonly bankRepository = inject(BankRepository);
  private readonly createTopup = inject(CreateTopupUseCase);
  private readonly _banks = signal<Bank[]>([]);
  private readonly _status = signal<'idle' | 'loading' | 'ready' | 'error'>('idle');
  private readonly _submitError = signal<AppError | null>(null);
  private readonly _invoice = signal<TopupInvoice | null>(null);
  // … _loadError, _submitting

  readonly banks = computed(() => selectableBanks(this._banks()));
  readonly status = this._status.asReadonly();
  readonly submitError = this._submitError.asReadonly();
  readonly invoice = this._invoice.asReadonly();

  async loadBanks(): Promise<void> { /* status → loading → ready | error */ }

  async submit({ bankCode, amount }: TopupFormValue): Promise<void> {
    if (!bankCode) return this._submitError.set({ type: 'validation', message: 'Choose a bank' });
    if (amount === null || !Number.isFinite(amount)) return this._submitError.set({ type: 'validation', message: 'Enter an amount' });
    // _submitting true → createTopup.execute → _invoice | _submitError
  }
}
```

## 10. Komponen

```ts
// features/topup/presentation/topup-page.ts (excerpt)
@Component({
  selector: 'app-topup-page',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [ReactiveFormsModule, ButtonDirective, CardComponent, InputDirective, LabelDirective, RadioDirective, OrDashPipe, EmptyStateComponent, ErrorStateComponent, SkeletonComponent],
  template: `
    @if (store.invoice(); as invoice) {
      <app-card title="Payment instructions"> … {{ invoice.amount | orDash: formatCurrency }} … </app-card>
    } @else {
      @switch (store.status()) {
        @case ('error') { @if (loadError(); as error) { <app-error-state [error]="error" /> } }
        @case ('ready') {
          <form [formGroup]="form" (ngSubmit)="submit()">
            @for (bank of store.banks(); track bank.code) {
              <label appLabel><input appRadio type="radio" formControlName="bankCode" [value]="bank.code" /> {{ bank.name | orDash }}</label>
            }
            <input appInput id="topup-amount" type="number" formControlName="amount" />
            @if (submitError(); as error) { <app-error-state [error]="error" /> }
            <button appButton type="submit" [disabled]="store.submitting()">Top up</button>
          </form>
        }
        @default { <app-skeleton class="h-5 w-40" /> … }
      }
    }
  `,
})
export class TopupPage implements OnInit {
  protected readonly store = inject(TopupStore);
  protected readonly form = new FormGroup({ bankCode: new FormControl<string | null>(null), amount: new FormControl<number | null>(null) });
  protected readonly submitError = computed(() => { const e = this.store.submitError(); return e ? toUiError(e) : null; });
  ngOnInit(): void { void this.store.loadBanks(); }
  protected submit(): void { void this.store.submit(this.form.getRawValue()); }
}
```

```ts
// shared/ui/button.ts — atom as an attribute directive on the native element
@Directive({ selector: 'button[appButton], a[appButton]', host: { '[class]': 'classes()' } })
export class ButtonDirective {
  readonly variant = input<'primary' | 'outline'>('primary');
  protected readonly classes = computed(() => `${BASE} ${VARIANTS[this.variant()]}`);
}
```

Komponen hanya menyuntik **store**, tidak pernah `HttpClient`, adapter, atau port (dipaksa lint).

## 11. Routing

`app.routes.ts`: `{ path: 'topup', loadChildren: () => import('@/features/topup/topup.routes').then((m) => m.TOPUP_ROUTES) }` dan `**` → redirect. Guard auth/role (bila ada) di `core/auth`. Alias `@/*` didefinisikan di `tsconfig.json` (`"paths": { "@/*": ["./src/*"] }`).

## 12. Testing

| Target | Cara |
|---|---|
| domain, mapper, use case, `formatOrDash` | unit test murni tanpa TestBed (wajib, test-first) |
| adapter HTTP | `TestBed` + `provideHttpClient()` + `provideHttpClientTesting()`; panggil method, `http.expectOne(url).flush(body, { status })` atau `.error(new ProgressEvent('error'))`, lalu `await` hasilnya |
| store | `TestBed` dengan `{ provide: BankRepository, useValue: fake }`, `{ provide: CreateTopupUseCase, useValue: { execute } }` |
| halaman | `TestBed.createComponent` + store asli di atas port palsu; `await fixture.whenStable()` |

Jebakan test: radio dengan `formControlName` + `[value]` — nilai disimpan di directive, **`input.value` di DOM tetap `"on"`**. Pilih radio lewat teks label, bukan selector `input[value=…]`.

## 13. Lint tambahan

```js
// eslint.config.js (package.json "type": "module")
export default defineConfig([
  globalIgnores(['dist/**', '.angular/**', 'out-tsc/**', 'guideline/**']),
  {
    files: ['**/*.ts'],
    extends: [eslint.configs.recommended, tseslint.configs.recommended, tseslint.configs.stylistic, angular.configs.tsRecommended],
    processor: angular.processInlineTemplates,
    rules: {
      '@angular-eslint/directive-selector': ['error', { type: 'attribute', prefix: 'app', style: 'camelCase' }],
      '@angular-eslint/component-selector': ['error', { type: 'element', prefix: 'app', style: 'kebab-case' }],
    },
  },
  { files: ['**/*.html'], extends: [angular.configs.templateRecommended, angular.configs.templateAccessibility] },
  ...boundaries,   // guideline/lint/eslint.boundaries.mjs — domain/kernel/shared-lib: no @angular/* or rxjs
]);
```

## 14. Jebakan & keputusan

Alasan desain: pola Angular yang sering gagal adalah "clean di kerangka, tidak di isi" — port/adapter dan DI sudah benar, tetapi ratusan use case hanya meneruskan, aturan bisnis menumpuk di service `helpers` global dan komponen ribuan baris, entity menyalin JSON backend, dan semua provider didaftarkan di satu file. Reference ini mencegahnya dengan: aturan bisnis sebagai fungsi murni di `domain/`, mapper wajib, use case hanya bila ada logika, store per fitur, dan provider per route.

- **Mock ikut bundle production** bila dipilih dengan flag runtime (`environment.useMock ? Mock : Http`) — juga dengan konstanta top-level. Terverifikasi: hanya `fileReplacements` yang membuangnya.
- **Jangan** `providedIn: 'root'` untuk adapter/store fitur — daftarkan di route fitur.
- **Jangan** menaruh aturan bisnis di service `helpers` global; jadikan fungsi murni di `domain/`.
- Komponen raksasa adalah gejala store/domain yang hilang — pecah sejak awal.
- `Promise<Result>` vs `Observable`: `Observable` hanya bila fitur butuh cancel/stream (SSE, polling, typeahead). Catat di `stack-profile.md`.
- `abstract class` vs `InjectionToken<Interface>`: default `abstract class`; token bila butuh beberapa implementasi untuk satu interface.
- **npm 10** bisa gagal memasang Vitest (`edgesOut`) — `ng new --skip-install`, lalu `npx npm@11 install`.
- Angular 22 menaikkan syarat Node; cek `npm view @angular/core@<versi> engines` sebelum upgrade.

# libreriaSauron-Frontend

Esta biblioteca le ayuda a implementar el cliente Keycloak-angular y keycloak-js en aplicaciones desarrolladas en Angular.

## Instalación en angular:

```
npm install keycloak-angular keycloak-js
```

## Función APP_INITALIZER:

   - Implementará por defecto una llamada al servidor de autenticación con los datos de url, nombre del REALM, y client id:

```
import { APP_INITIALIZER, NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { KeycloakAngularModule, KeycloakService } from 'keycloak-angular';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
 
function initializeKeycloak(keycloak: KeycloakService) {
  return () =>
    keycloak.init({
      config: {
        url: 'http://localhost:8080/auth',
        realm: 'your-realm',
        clientId: 'your-client-id',
      },
      initOptions: {
        onLoad: 'check-sso',
        silentCheckSsoRedirectUri:
          window.location.origin + '/assets/silent-check-sso.html',

          // Excluye las rutas publicas ejemplo:
          // bearerExcludedUrls: ['/assets', '/clients/public'],
      },
    });
}
 
@NgModule({
  declarations: [AppComponent],
  imports: [AppRoutingModule, BrowserModule, KeycloakAngularModule],
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: initializeKeycloak,
      multi: true,
      deps: [KeycloakService],
    },
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

## Función AUTHGUARD:

   - Protege las rutas autenticadas en la aplicación, pudiendo acceder a ellas en función de los roles de la API.

```
import { Injectable } from '@angular/core';
import {
  ActivatedRouteSnapshot,
  Router,
  RouterStateSnapshot,
} from '@angular/router';
import { KeycloakAuthGuard, KeycloakService } from 'keycloak-angular';
 
@Injectable({
  providedIn: 'root',
})
export class AuthGuard extends KeycloakAuthGuard {
  constructor(
    protected readonly router: Router,
    protected readonly keycloak: KeycloakService
  ) {
    super(router, keycloak);
  }
 
  public async isAccessAllowed(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ) {
    // Fuerza la autenticacon al acceder al recurso si no esta atenticado
    if (!this.authenticated) {
      await this.keycloak.login({
        redirectUri: window.location.origin + state.url,
      });
    }
 
    // Define los roles necesarios para acceder a la ruta
    const requiredRoles = route.data.roles;
 
    // Permite que el usuario continue si no se requieren roles adicionales para acceder a la ruta.
    if (!(requiredRoles instanceof Array) || requiredRoles.length === 0) {
      return true;
    }
 
    // Permite continuar al usuario si dispone de todos los roles requeridos
    return requiredRoles.every((role) => this.roles.includes(role));
  }
}
```
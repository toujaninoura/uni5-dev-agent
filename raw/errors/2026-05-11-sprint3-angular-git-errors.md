---
Source: dev session 2026-05-11
Collected: 2026-05-11
Published: 2026-05-11
---

# Erreurs rencontrées — Sprint 3 (Angular + Git)

## RouterTestingModule deprecated in Angular 17
Erreur : Tests Jasmine echouent avec "TypeError: Cannot read properties of undefined (reading 'root')" quand RouterLink est present dans le composant
Cause : RouterTestingModule (deprecated depuis Angular 16) conflicte avec { provide: Router, useValue: routerSpy } — le spy n'a pas de propriete 'root' attendue par RouterLink
Fix : Remplacer RouterTestingModule.withRoutes([]) par provideRouter([]) + provideLocationMocks() dans TestBed. Ne pas fournir de spy Router — injecter le vrai Router puis spyOn(router, 'navigate')
Prevention : Jamais RouterTestingModule pour Angular 17. Pattern : provideRouter([]) + provideLocationMocks() + spyOn(TestBed.inject(Router), 'navigate')

## TS2345 — signature mismatch apres changement de type sur un service
Erreur : Build Angular echoue avec TS2345 "Argument of type { email, password } is not assignable to parameter of type RegisterRequest"
Cause : AuthService.register() signature changee de AuthRequest vers RegisterRequest mais anciens appelants (tests, LoginComponent) passes sans mise a jour
Fix : Mettre a jour TOUS les appelants lors du changement de signature : login.component.ts, auth.service.spec.ts, login.component.spec.ts
Prevention : Avant de changer la signature d'un service, chercher tous les appelants avec grep. Changer signature + tous les appelants en un seul commit

## isLoading non remis a false apres succes dans un composant Angular
Erreur : Bouton submit reste desactive apres une requete reussie (si composant reste visible ou si navigation echoue)
Cause : isLoading = false absent du callback next — seulement present dans le callback error
Fix : Toujours reset isLoading dans les deux callbacks : next: () => { this.isLoading = false; this.router.navigate(...); } et error: () => { this.isLoading = false; ... }
Prevention : Pattern systematique — isLoading = false TOUJOURS dans next ET dans error. Ne jamais assumer que la navigation detruira le composant

## GitHub PR non mergeable — branche orpheline
Erreur : gh pr merge echoue avec "merge commit cannot be cleanly created" — GitHub suggere de merger origin/feat/issue-1-setup-ntier
Cause : Branche feature tres ancienne (feat/issue-1-setup-ntier) jamais supprimee — GitHub calcule un ancetre commun errone et pense qu'il y a un conflit non resolu
Fix : Merger la feature branch directement en local (git merge feat/xxx --no-ff) puis git push origin main. Fermer la PR GitHub manuellement avec un commentaire.
Prevention : Supprimer les branches remote immediatement apres merge. Ne jamais laisser de branches orphelines sur GitHub (git push origin --delete nom-branche)

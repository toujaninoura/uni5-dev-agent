# Source brute — Scope imbrique dans DatabaseSeeder
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : SeedAsync recoit un IServiceProvider deja scope depuis Program.cs, puis recrée un scope interne avec CreateScope() -> scope dispose trop tot -> ObjectDisposedException potentielle sur AppDbContext
Contexte: DatabaseSeeder.SeedAsync appelé depuis Program.cs avec scope.ServiceProvider
Fix     : Utiliser directement serviceProvider.GetRequiredService<AppDbContext>() sans CreateScope()

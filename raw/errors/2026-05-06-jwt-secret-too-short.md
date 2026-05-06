# Source brute — Erreur JWT Secret trop court
Date    : 2026-05-06
Projet  : contacts-app + task-manager
Erreur  : IDX10720: Unable to create KeyedHashAlgorithm for algorithm HS256,
          the key size must be greater than 256 bits, key has 248 bits.
Contexte: Appel POST /api/v1/auth/login ou /register
Fix     : dotnet user-secrets set "JWT:Secret" "TaskManager2026_SuperSecretKey_ForHS256!!"

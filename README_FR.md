# IPSEN Container Tracking — prototype indépendant

## Ce qui est inclus
- `index.html` : interface responsive (ordinateur + téléphone), tableau de bord, registre, alertes, fiche conteneur, timeline.
- `config.js` : configuration Supabase publique.
- `supabase/schema.sql` : tables dédiées + politiques RLS.
- `supabase/functions/carrier-sync/index.ts` : squelette sécurisé pour une future connexion à une API transporteur.

## Démarrage en mode démonstration
Ouvrir `index.html` dans un navigateur connecté à Internet. Sans configuration Supabase, le site affiche des données fictives de démonstration. Les modifications de démonstration ne sont pas persistantes.

## Connexion Supabase
1. Créer un projet Supabase indépendant ou choisir le projet IPSEN autorisé.
2. Exécuter `supabase/schema.sql` dans SQL Editor.
3. Dans `Authentication > Users`, créer un utilisateur IPSEN.
4. Ajouter son profil dans `public.profiles` (table déjà utilisée par le portail IPSEN). Exemple :
   ```sql
   insert into public.profiles (id, role, client_key, client_name)
   values ('UUID_UTILISATEUR', 'staff', null, 'Équipe Logistique IPSEN')
   on conflict (id) do update set role='staff', client_key=null, client_name='Équipe Logistique IPSEN';
   ```
   Pour un administrateur, utiliser `role='admin'`.
5. Copier Project URL et anon/publishable key dans `config.js`.
6. Héberger le dossier sur Netlify, Vercel ou un hébergeur HTTPS statique.

> Ne jamais placer la clé `service_role` dans `config.js` ou dans le navigateur.

## Étapes suivies
Préavis reçu → Arrivée au port → Déchargement navire → Formalités douanières → Bon à enlever → Sortie du port → En route vers IPSEN → Arrivée IPSEN.

Les événements manuels sont utilisables immédiatement une fois Supabase configuré. Le système conserve l'heure, la source et les notes de chaque événement.

## Suivi automatique transporteur
Le suivi automatique réel n'est pas activé par défaut. Il faut :
- connaître les compagnies maritimes réellement utilisées ;
- obtenir un accès API officiel / partenaire (clé et documentation) ;
- déployer une fonction serveur Supabase Edge Function ;
- mapper les événements du fournisseur vers les étapes IPSEN ;
- configurer une fréquence de synchronisation (cron) et vérifier les conditions contractuelles.

Le fichier `carrier-sync/index.ts` est un squelette d'intégration, pas un connecteur opérationnel. Ne jamais exposer les clés API transporteur dans le navigateur.

## Règle d'alerte
- moins de 2 jours : dans le délai ;
- à partir de 2 jours et jusqu'à 3 jours : à surveiller ;
- plus de 3 jours : retard.
Le délai est calculé entre `notice_at` et `arrived_at` (ou l'heure actuelle si le conteneur n'est pas encore arrivé).

## Important
Ceci est un prototype de démarrage. Avant la production, tester les rôles, RLS, fuseau horaire, doublons de numéros, exactitude des événements, et faire valider l'usage des données par IPSEN.

# TD
## *EXERCICE 1 - PARTIE 1*

### *1. Implémentation des classes*
java
// Classe mère
class Personnel {
    private double salaire;
    
    public Personnel(double salaire) {
        this.salaire = salaire;
    }
    
    public double getSalaire() {
        return salaire;
    }
    
    public void setSalaire(double salaire) {
        this.salaire = salaire;
    }
}

// Classe Département
class Departement {
    private String nomD;
    private Gerant gerant; // Relation one-to-one
    
    public Departement(String nomD, Gerant gerant) {
        this.nomD = nomD;
        this.gerant = gerant;
        // Affection bidirectionnelle
        gerant.setDepartement(this);
    }
    
    public String getNomD() {
        return nomD;
    }
    
    public Gerant getGerant() {
        return gerant;
    }
    
    public void setGerant(Gerant gerant) {
        this.gerant = gerant;
    }
}

// Classe Gérant
class Gerant extends Personnel {
    private String specialite;
    private Departement departement; // Relation one-to-one
    
    public Gerant(double salaire, String specialite) {
        super(salaire);
        this.specialite = specialite;
    }
    
    public String getSpecialite() {
        return specialite;
    }
    
    public Departement getDepartement() {
        return departement;
    }
    
    public void setDepartement(Departement departement) {
        this.departement = departement;
    }
}

// Classe Assistant
class Assistant extends Personnel {
    private int anneeDebu;
    
    public Assistant(double salaire, int anneeDebu) {
        super(salaire);
        this.anneeDebu = anneeDebu;
    }
    
    public int getAnneeDebu() {
        return anneeDebu;
    }
    
    public void setAnneeDebu(int anneeDebu) {
        this.anneeDebu = anneeDebu;
    }
}


*Types de relations :*
- Departement ↔️ Gerant : *One-to-One*
- Gerant hérite de Personnel
- Assistant hérite de Personnel

### *2. Création d'objets*
java
public class Main {
    public static void main(String[] args) {
        // Création de Gérants
        Gerant gerant1 = new Gerant(12000, "Informatique");
        Gerant gerant2 = new Gerant(15000, "Ressources Humaines");
        
        // Création d'Assistants
        Assistant assistant1 = new Assistant(8000, 2020);
        Assistant assistant2 = new Assistant(7500, 2021);
        
        // Création de Départements (affectés à des gérants)
        Departement dep1 = new Departement("IT", gerant1);
        Departement dep2 = new Departement("RH", gerant2);
    }
}


---

## *EXERCICE 1 - PARTIE 2*

### *1. Méthode pour ajouter un département*
java
public void ajouterDepartement(Set<Departement> lesDeps, Departement nouveauDep) {
    // Vérifier l'unicité du nom
    boolean nomExiste = lesDeps.stream()
        .anyMatch(dep -> dep.getNomD().equals(nouveauDep.getNomD()));
    
    if (nomExiste) {
        throw new IllegalArgumentException("Un département avec ce nom existe déjà");
    }
    
    // Vérifier qu'il a un gérant
    if (nouveauDep.getGerant() == null) {
        throw new IllegalArgumentException("Un département doit avoir un gérant");
    }
    
    lesDeps.add(nouveauDep);
}


### *2. Méthode statique pour supprimer un département*
java
public static void supprimerDepartement(Set<Departement> lesDeps, String nomDep) {
    // Trouver le département
    Departement depASupprimer = lesDeps.stream()
        .filter(dep -> dep.getNomD().equals(nomDep))
        .findFirst()
        .orElse(null);
    
    if (depASupprimer != null) {
        // Traiter la relation avec le gérant
        Gerant gerant = depASupprimer.getGerant();
        if (gerant != null) {
            gerant.setDepartement(null); // Libérer la relation
        }
        
        // Supprimer le département
        lesDeps.remove(depASupprimer);
    }
}


### *3. Regrouper assistants par année de début*
java
public Map<Integer, Long> regrouperAssistantsParAnnee(List<Personnel> lesPrs) {
    return lesPrs.stream()
        .filter(p -> p instanceof Assistant) // Filtrer seulement les assistants
        .map(p -> (Assistant) p) // Caster en Assistant
        .collect(Collectors.groupingBy(
            Assistant::getAnneeDebu,
            Collectors.counting() // Compter par année
        ));
}


### *4. Regrouper gérants par spécialité (salaire < 10000)*
java
public Map<String, List<Gerant>> regrouperGerantsParSpecialite(List<Personnel> lesPrs) {
    return lesPrs.stream()
        .filter(p -> p instanceof Gerant) // Filtrer les gérants
        .map(p -> (Gerant) p) // Caster en Gerant
        .filter(g -> g.getSalaire() < 10000) // Salaire < 10000
        .collect(Collectors.groupingBy(
            Gerant::getSpecialite
        ));
}


---

## *EXERCICE 3 - Java Streams*

java
public List<Produit> traiterProduits(Set<Produit> produits) {
    return produits.stream()
        // Appliquer réduction de 15% sur électronique < 2000
        .peek(p -> {
            if (p.getCategorie().equals("Electronique") && p.getPrix() < 2000) {
                p.setPrix(p.getPrix() * 0.85); // Réduction de 15%
            }
        })
        // Modifier les IDs selon la catégorie
        .peek(p -> {
            if (!p.getCategorie().equals("Librairie") && !p.getCategorie().equals("Jardin")) {
                String prefix = p.getCategorie().substring(0, 3).toUpperCase();
                p.setId(prefix + "-" + p.getId());
            }
        })
        // Collecter dans une liste accessible par indices
        .collect(Collectors.toList());
}


*Classe Produit :*
java
class Produit {
    private String id;
    private String nom;
    private double prix;
    private String categorie;
    private int anneeFabric;
    
    // Constructeur, getters, setters
    public Produit(String id, String nom, double prix, String categorie, int anneeFabric) {
        this.id = id;
        this.nom = nom;
        this.prix = prix;
        this.categorie = categorie;
        this.anneeFabric = anneeFabric;
    }
    
    // Getters et setters
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getNom() { return nom; }
    public double getPrix() { return prix; }
    public void setPrix(double prix) { this.prix = prix; }
    public String getCategorie() { return categorie; }
    // ...
}


---

## *EXERCICE 4 - Exceptions*

java
public class MainException {
    public static void main(String[] args) {
        Etudiant[] students = new Etudiant[5];
        // Initialisation partielle pour simuler null
        students[0] = new Etudiant("Alice", 15.5);
        students[2] = new Etudiant("Bob", 16.0);
        // students[1], [3], [4] restent null
        
        try {
            Etudiant best = students[0];
            
            for(Etudiant e : students) {
                // Vérification null avant accès
                if(e != null) {
                    if(e.moy > best.moy) {
                        best = e;
                    }
                }
            }
            
            System.out.println(" " + best.moy);
            
        } catch (NullPointerException e) {
            System.out.println("Erreur Null pointer trouvée");
        }
    }
}

class Etudiant {
    String nom;
    double moy;
    
    public Etudiant(String nom, double moy) {
        this.nom = nom;
        this.moy = moy;
    }
}


---

## *EXERCICE 5 - ThreadPoolExecutor*

### *1. Critères avant d'adopter ThreadPoolExecutor*
- Volume de tâches à exécuter
- Nature des tâches (CPU-intensive vs I/O-intensive)
- Besoin de contrôle sur le nombre de threads
- Nécessité de réutiliser les threads pour éviter le coût de création
- Gestion de la file d'attente des tâches

### *2. Déclaration avec 7 threads*
java
ExecutorService executor = Executors.newFixedThreadPool(7);


### *3. Classe qui hérite de Thread*
- Une tâche exécutable
- Doit implémenter Runnable ou étendre Thread
- Représente une unité de travail pour le pool

### *4. Utilité de run()*
- Méthode contenant le code à exécuter dans le thread
- Appelée automatiquement quand le thread démarre
- Ne doit pas être appelée directement (utiliser start())

### *5. Utilité de submit()*
- Soumet une tâche au pool d'exécution
- Retourne un Future pour suivre l'exécution
- Permet de récupérer le résultat ou l'exception

### *6. File d'attente (Queue)*
- Stocke les tâches en attente d'exécution
- Gère le débordement quand tous les threads sont occupés
- Peut être configurée avec différentes politiques (FIFO, priorité)

### *7. Problèmes possibles*
- Deadlock si mauvaise synchronisation
- Starvation si certaines tâches attendent indéfiniment
- Fuite mémoire si threads non fermés
- Sur-utilisation CPU si trop de threads

### *8. Collection Future<?>*
- Stocke les résultats futurs des tâches
- Permet d'attendre la fin de toutes les tâches
- Facilite la gestion des exceptions
java
List<Future<?>> futures = new ArrayList<>();
futures.add(executor.submit(task));


---

## *EXERCICE 6 - Synchronisation*

### *1. Importance de la synchronisation*
- Évite les conditions de course (race conditions)
- Garantit la cohérence des données partagées
- Préserve l'intégrité des opérations critiques

### *2. Problème résolu*
- Accès concurrent à des ressources partagées
- Modifications simultanées de variables
- Lecture de données inconsistantes

### *3. Utilité de Lock*
- Contrôle plus fin que synchronized
- Permet d'essayer d'obtenir le verrou (tryLock())
- Possibilité de timeout
java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // section critique
} finally {
    lock.unlock();
}


### *4. Utilité de synchronized*
- Verrou intrinsèque sur l'objet
- Plus simple à utiliser que Lock
- Libération automatique en sortant du bloc

### *5. Détecter problème de synchronisation*
- Tests avec accès concurrent
- Analyse de code statique
- Utilisation d'outils comme ThreadSanitizer
- Observation de résultats incohérents

### *6. Points clés pour la synchronisation*
- Identifier les sections critiques
- Minimiser le temps de verrouillage
- Éviter les deadlocks (ordre constant des verrous)
- Utiliser des collections thread-safe quand possible

### *7. Section critique*
- Partie du code qui accède à des ressources partagées
- Doit être exécutée par un seul thread à la fois
- Doit être protégée par un mécanisme de synchronisation

### *8. start() et join()*
- start() : démarre l'exécution du thread
- join() : attend la fin de l'exécution du thread
java
Thread t = new Thread(task);
t.start();  // Démarre le thread
t.join();   // Attends sa fin


---

## *EXERCICE 7 - Threads (SQL)*

### *1a. Synchronized*
java
public synchronized static void execQuery(String query, Connection con) {
    // Code existant...
}


### *1b. Lock*
java
public class Operations {
    private static final Lock lock = new ReentrantLock();
    
    public static void execQuery(String query, Connection con) {
        lock.lock();
        try {
            // Code existant...
        } finally {
            lock.unlock();
        }
    }
}


### *2. Exécution avec attente*
java
public class MainEx {
    public static void main(String[] args) {
        // ... code de connexion ...
        
        TaskExecQuery task1 = new TaskExecQuery(connection, "SELECT * FROM membre");
        TaskExecQuery task2 = new TaskExecQuery(connection, "SELECT hm FROM membre");
        
        Thread th1 = new Thread(task1);
        Thread th2 = new Thread(task2);
        
        // th2 doit attendre th1
        th1.start();
        try {
            th1.join();  // th2 attend la fin de th1
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        th2.start();
    }
}


### *3. Utilités*
- *DriverManager* : Gère les connexions aux bases de données
- *jdbc:mysql://* : URL de connexion JDBC pour MySQL

---

## *EXERCICE 7 - Hibernate*

### *1. Annotations de la classe Département*
- @Entity : Déclare la classe comme entité persistante
- @Table(name = "departement") : Mappe la classe à la table SQL
- @Id : Identifie la clé primaire
- @GeneratedValue : Stratégie de génération automatique
- @Column : Mappe un attribut à une colonne
- @OneToMany : Définit une relation 1-N avec Livre

### *2. Classe Livre complète*
java
@Entity
@Table(name = "livre")
public class Livre {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "idL")
    private Long idL;
    
    @Column(name = "titre")
    private String titre;
    
    @Column(name = "auteur")
    private String auteur;
    
    @ManyToOne
    @JoinColumn(name = "idDep")
    private Departement departement;
    
    // Constructeur, getters, setters
    public Livre() {}
    
    public Long getIdL() { return idL; }
    public void setIdL(Long idL) { this.idL = idL; }
    // ... autres getters/setters
}


### *3. cascade = CascadeType.ALL*
- Applique toutes les opérations (persist, merge, remove, refresh, detach) aux entités enfants
- Quand on sauvegarde/supprime un Département, tous ses Livres sont affectés

### *4. Rôles dans DepartementDAO*
- *Transaction* : Unité de travail atomique
- *beginTransaction()* : Démarre une transaction
- *createQuery()* : Crée une requête HQL
- *commit()* : Valide la transaction
- *SessionFactory* : Fabrique de sessions, coûteuse à créer

### *5. Méthode updateDepartment*
java
public void updateDepartment(Departement department) {
    Session session = sessionFactory.openSession();
    Transaction tx = null;
    try {
        tx = session.beginTransaction();
        session.update(departement);
        tx.commit();
    } catch (Exception e) {
        if (tx != null) tx.rollback();
        throw e;
    } finally {
        session.close();
    }
}


### *6. Concepts Java*
- *Jakarta* : Successeur de Java EE pour l'enterprise
- *Persistance des données* : Sauvegarde des objets en base de données
- *ORM (Object-Relational Mapping)* : Mapping entre objets Java et tables SQL

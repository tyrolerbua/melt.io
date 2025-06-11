<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Meine schöne Webseite</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f0f4f8;
      margin: 0;
      padding: 0;
    }
    header {
      background-color: #0077cc;
      color: white;
      padding: 20px;
      text-align: center;
    }
    nav {
      background-color: #005fa3;
      padding: 10px;
      text-align: center;
    }
    nav a {
      color: white;
      text-decoration: none;
      margin: 0 15px;
      font-weight: bold;
    }
    main {
      padding: 20px;
    }
    img {
      max-width: 100%;
      height: auto;
      border-radius: 8px;
    }
    button {
      background-color: #0077cc;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin-top: 20px;
    }
    button:hover {
      background-color: #005fa3;
    }
    footer {
      background-color: #e0e0e0;
      text-align: center;
      padding: 10px;
      margin-top: 30px;
    }
  </style>
</head>
<body>

  <header>
    <h1>Willkommen auf meiner Webseite</h1>
    <p>Schön, dass du da bist!</p>
  </header>

  <nav>
    <a href="#">Start</a>
    <a href="#">Über mich</a>
    <a href="#">Kontakt</a>
  </nav>

  <main>
    <h2>Über mich</h2>
    <p>Ich bin ein Webentwickler und liebe es, schöne Webseiten zu bauen.</p>
    <img src="https://via.placeholder.com/600x300" alt="Beispielbild">
    <br>
    <button onclick="alert('Danke für deinen Besuch!')">Klick mich</button>
  </main>

  <footer>
    &copy; 2025 Meine Webseite
  </footer>

</body>
</html>

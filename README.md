--------------------------------------------------------------------
1. Stored XSS in Comment System
--------------------------------------------------------------------
Vulnerable Code:
--------------------------------------------------------------------
<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $comment = $_POST["comment"];
    // Vulnerable: Directly inserting user input into HTML
    echo "<p>Comment: " . $comment . "</p>";
}
?>
<form method="post">
    <textarea name="comment"></textarea>
    <button type="submit">Submit</button>
</form>
--------------------------------------------------------------------
Why It’s Vulnerable?
--------------------------------------------------------------------
•	The user input ($comment) is directly echoed in HTML without sanitization.
•	An attacker can submit <script>alert('XSS')</script> which will execute JavaScript on the page.
--------------------------------------------------------------------
Secure Code:
--------------------------------------------------------------------
<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $comment = htmlspecialchars($_POST["comment"], ENT_QUOTES, 'UTF-8');
    echo "<p>Comment: " . $comment . "</p>";
}
?>
<form method="post">
    <textarea name="comment"></textarea>
    <button type="submit">Submit</button>
</form>
--------------------------------------------------------------------
Fix Explanation:
•	htmlspecialchars() converts special characters (<, >, " etc.) into HTML entities, preventing script execution.
--------------------------------------------------------------------
2. Reflected XSS in Search Feature
--------------------------------------------------------------------
Vulnerable Code:
--------------------------------------------------------------------
<?php
$search = $_GET['query'];
echo "<p>Search results for: " . $search . "</p>";
?>
<form method="get">
    <input type="text" name="query">
    <button type="submit">Search</button>
</form>
--------------------------------------------------------------------
Why It’s Vulnerable?
--------------------------------------------------------------------
•	User input is directly echoed into the page.
•	An attacker can send a link like:
http://example.com/search.php?query=<script>alert('XSS')</script>
The JavaScript will execute when the page loads.
--------------------------------------------------------------------
Secure Code:
--------------------------------------------------------------------
<?php
$search = htmlspecialchars($_GET['query'], ENT_QUOTES, 'UTF-8');
echo "<p>Search results for: " . $search . "</p>";
?>
<form method="get">
    <input type="text" name="query">
    <button type="submit">Search</button>
</form>
--------------------------------------------------------------------
Fix Explanation:
--------------------------------------------------------------------
•	htmlspecialchars() prevents script execution by encoding characters like < and >.
--------------------------------------------------------------------
3. XSS in JavaScript Context
--------------------------------------------------------------------
Vulnerable Code:
--------------------------------------------------------------------
<?php
$username = $_GET['name'];
?>
<script>
    var name = "<?php echo $username; ?>";
    alert("Welcome, " + name);
</script>
--------------------------------------------------------------------
Why It’s Vulnerable?
--------------------------------------------------------------------
•	User input is injected directly into JavaScript.
•	An attacker can send:
http://example.com/welcome.php?name=";alert('XSS');//
This breaks the script and executes JavaScript.
--------------------------------------------------------------------
Secure Code:
--------------------------------------------------------------------
<?php
$username = htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
?>
<script>
    var name = "<?php echo addslashes($username); ?>";
    alert("Welcome, " + name);
</script>
--------------------------------------------------------------------
Fix Explanation:
--------------------------------------------------------------------
•	htmlspecialchars() prevents HTML injection.
•	addslashes() prevents JavaScript string manipulation by escaping quotes.
--------------------------------------------------------------------
4. XSS in Image Upload Preview Feature
--------------------------------------------------------------------
Vulnerable Code:
--------------------------------------------------------------------
<input type="file" id="upload">
<img id="preview">
<script>
    document.getElementById('upload').addEventListener('change', function(event) {
        var file = event.target.files[0];
        document.getElementById('preview').src = URL.createObjectURL(file);
    });
</script>
--------------------------------------------------------------------
Why It’s Vulnerable?
--------------------------------------------------------------------
•	If an attacker uploads a malicious SVG file containing JavaScript, it can execute when previewed.
--------------------------------------------------------------------
Secure Code:
--------------------------------------------------------------------
<input type="file" id="upload" accept="image/png, image/jpeg">
<img id="preview">
<script>
    document.getElementById('upload').addEventListener('change', function(event) {
        var file = event.target.files[0];
        var allowedTypes = ["image/png", "image/jpeg"];
        if (allowedTypes.includes(file.type)) {
            document.getElementById('preview').src = URL.createObjectURL(file);
        } else {
            alert("Invalid file type!");
        }
    });
</script>
--------------------------------------------------------------------
Fix Explanation:
--------------------------------------------------------------------
•	Restricts allowed file types to image/png and image/jpeg.
•	Prevents malicious SVG files from executing JavaScript.
--------------------------------------------------------------------
5. XSS in an Anchor Tag (URL Injection)
--------------------------------------------------------------------
Vulnerable Code:
--------------------------------------------------------------------
<?php
$link = $_GET['url'];
echo '<a href="' . $link . '">Click Here</a>';
?>
Why It’s Vulnerable?
•	An attacker can craft a URL like:
http://example.com?url=javascript:alert('XSS')
Clicking the link will execute JavaScript.
--------------------------------------------------------------------
Secure Code:
--------------------------------------------------------------------
<?php
$link = filter_var($_GET['url'], FILTER_SANITIZE_URL);
if (filter_var($link, FILTER_VALIDATE_URL)) {
    echo '<a href="' . htmlspecialchars($link, ENT_QUOTES, 'UTF-8') . '">Click Here</a>';
} else {
    echo "Invalid URL!";
}
?>
--------------------------------------------------------------------
Fix Explanation:
--------------------------------------------------------------------
•	FILTER_SANITIZE_URL removes unwanted characters.
•	FILTER_VALIDATE_URL ensures the input is a valid URL.
•	htmlspecialchars() prevents script execution.


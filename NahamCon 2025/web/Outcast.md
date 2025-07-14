<img width="1339" height="638" alt="image" src="https://github.com/user-attachments/assets/9014a9fb-d467-4ce5-b791-0ca7568b25dd" />

# Initial Recon
While inspecting the html of the app, I found a comment mentioning a hidden `/test` directory. Navigating to `/test` revealed a "**API caller**":

<img width="1245" height="615" alt="image" src="https://github.com/user-attachments/assets/e39e88b1-8fb4-4d80-9a67-b31725e2141e" />

Theres also a admin login page at the `/login` directory, we will use this later.
<img width="799" height="698" alt="image" src="https://github.com/user-attachments/assets/44184237-600e-44c4-aca5-5a92b89d530c" />

# Looking at the /test directory.
On the `/test` directory there was a button labeled "**Download template code**", when clicked, it revealed the backend PHP source code for the "**API caller**".
```php
$id = $id;
$this->path_tmp = $path_tmp;
}

public function __call($apiMethod, $data = array())
{
    $url = $this->url . $apiMethod;
    $data['id'] = $this->id;

    foreach ($data as $k => &$v) {
        if (($v) && (is_string($v)) && str_starts_with($v, '@')) {
            $file = substr($v, 1);
            if (str_starts_with($file, $this->path_tmp)) {
                $v = file_get_contents($file);
            }
        }

        if (is_array($v) || is_object($v)) {
            $v = json_encode($v);
        }
    }

    // Call the API server using the given configurations
    $ch = curl_init($url);
    curl_setopt_array($ch, array(
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => $data,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => array('Accept: application/json'),
    ));

    $response = curl_exec($ch);
    $error = curl_error($ch);
    curl_close($ch);

    if (!empty($error)) {
        throw new Exception($error);
    }

    return $response;
}
```
# Observations
- File Inclusion: If a parameter starts with `@` (for example, `@/etc/passwd`), it is treated as a file path. The application then reads the contents of the file and includes it in the API request.
- Path Bypass: Theres a check to make sure the file path starts with `/tmp/`, but this is only a string based check it doesn’t actually verify the file location after resolving the path. By using `../` for directory traversal (e.g., `/tmp/../flag.txt`), we can escape the tmp directory and bypass this check to read files outside of it.

# Exploiting the File Inclusion Vulnerability
## Finding the flag
To read the flag, I had to figure out the correct file path. At first, I had no clue where the flag might be. I started throwing random file paths into the API and noticed that if a file didn’t exist, the API would throw a "file not found" error. If it did, nothing happened.

After a few attempts, I found that the path `/tmp/../flag.txt` did not trigger an error, while other random file paths did.

## Reflecting the flag
To reflect the flag on the page, I had to send a request that would include the contents of the file in the response. Initially, I thought about just sending the file path as a parameter, but I realized that wouldn't work since the file contents needed to be reflected somewhere.

After some more digging, I figured out that the `/login` page reflected the username in the html after a failed login attempt. This was the perfect place to inject the contents of flag.txt.

# Exploit
I crafted the request as follows:

- API Method: The API method had to be set to `../login`. This escapes the `/api` directory and sends the request to the login page.

- Parameters: `username=@/tmp/../flag.txt&password=aaa`

Heres what happens:

- The @ symbol indicates that the username value is a file path.

- The path `/tmp/../flag.txt` resolves to `/flag.txt` due to the directory traversal (../).

- flag.txt is read by the backend and included in the username field.


<img width="881" height="438" alt="image" src="https://github.com/user-attachments/assets/e30e655f-00dd-4528-b5c8-880a433755c4" />

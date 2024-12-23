# autoforti
Auto (re)connect to Fortinet VPNs with 2FA

Do you also hate the official Forticlient for macOS? Do you want a client that remembers your credentials, looks up the 2FA code from your email account, and automatically reconnects when the connection is disrupted? Look no further...

## Supported OSs
- macOS (tested)
- Linux (probably...)

## Instructions (for macOS)
### Requirements
- `openforticlient` (`brew install openforticlient`)
- `uv` (`brew install uv`)

`uv` is not technically required, it's just a nice way to run a self-contained Python script, and it's what I have used.

### Setup
#### Configure `openforticlient`
Find the default location of `openforticlient`'s configuration file by running `openfortivpn --help`.
On the homebrew version on Apple Silicon it's `/opt/homebrew/etc/openfortivpn/openfortivpn/config`.

Create the file if it doesn't exist, and fill in these values:
```conf
host =
port =
username =
password =
```

#### Root permission for `openforticlient`
Running `openforticlient` requires you to be `root`. The script runs `sudo openforticlient` non-interactively, so you can't enter your password in the terminal.
Technically I can have the script relinquish control to the terminal for entering the `sudo` password, but there are nicer solutions.
- You can run the script as root
- You can configure `sudo` to use touch id authentication
```sh
sed -e 's/^#auth/auth/' /etc/pam.d/sudo_local.template | sudo tee /etc/pam.d/sudo_local
```
- You can configure the script to automatically enter your `sudo` password by uncommenting and setting `SUDO_PASSWORD = 'sudo_password'` on line 20, and also uncommenting lines 48-49
- You can configure `sudo` to not require authentication when running `openfortivpn`
```sh
sudo visudo --file=/etc/sudoers.d/openfortivpn
```
Add the following line:
```
%admin ALL=(root) NOPASSWD: /opt/homebrew/bin/openfortivpn
```
#### Execute permission for the script
Download `autoforti` from this repo, and place it somewhere in your `$PATH`

Run `chmod +x /path/to/autoforti`

### Configuration (for gmail)
Create an app password [here](https://myaccount.google.com/apppasswords).

Open the script file in your favorite text editor and set `IMAP_LOGIN` to your email address and `IMAP_PASSWORD` to the app password.

### Other email providers
Some changes may need to be made for other email providers. You may need to change the port or disable TLS.

The script archives the 2FA email after the code has been extracted.

For other email providers the folder is probably different.

You may want to delete it instead:
```python
mailbox.delete(msg.uid)
```
For gmail this code seems to be necessary to delete:
```python
mailbox.move(msg.uid, "[Gmail]/Bin")
```

### Running
```sh
autoforti
```

# 1p -- A quick command-line interface over 1Password

1Password is awesome, but its best interface, the desktop GUI, is reserved to
macOS and Windows users. Sad! Fortunately, we Linux/Keyboard gnomes can get by
as long as we can have one reliable interface: a CLI tool. AgileBits, the folk
that make 1Password, provide such a tool, but it is *clunky*. This project
aims at restoring CLI usability to the 1Password service.

## Shoehorning 1Password to a shell interface

Desktop interfaces to 1Password offer a rich object kit to store in the secure
database: logins, but also all sorts of accounts, numbers, passports, anything
everybody agrees not to put on Facebook, and then some. Such zoological
complexity of information to manage goes against CLI usability. So let's
reduce this feature set to what can cover two use cases:

1. Store and fetch username/password pairs for named services.
2. Store useful notes containing secrets regarding a named service.

The 1p program provided here covers these two use cases, aiming for simplicity
and convenience.

## Installing

1. Clone this repository.
1. Run the install script:
    ```sh
    sudo ./install
    ```

From then on, the `1p` tool is available for all users of the system.

### Initial setup

The first time you run `1p`, it will ask for four elements of information so
as to fast-track future 1Password logons:

1. 1Password domain URL;
1. E-mail address used as 1Password username;
1. So-called *secret key* provided by 1Password when you set up your account;
1. Your master password.

You can get the first three items from your 1Password *emergency kit*, a PDF
with information to set up 1Password from scratch, which 1Password gave you
when you first set it up. You will also find it on the preference dialogs of
the desktop interfaces. There is likely a similar way to eke it out of the
browser interface.

This information will be encrypted using GPG (using a key generated
on-the-fly, for which you supply an ad hoc password) and stored on disk. When
you next use `1p`, your system will ask that you input the password to unlock
this new GPG key. On most systems, you should be given an option to store this
GPG key into a a keyring or agent system, reducing the password input chore to
your keyring/agent automatic lock policy.

## Storing and fetching username/password pairs

This is the simplest use case for 1Password. For instance, here's how to store
your Google password:

```sh
$ 1p google google_password_123%
```

(No, boss, it's not my real Google password.) Here, `google` is the name I
give to the service, for retrieving the password later. You may use a name
with spaces, using your shell quoting rules:

```sh
$ 1p "Alternate Facebook" 'Str0ngPa$$w0rdN0T!'
```

Honestly, entering passwords on the command line is a rather bad idea: not
only does it require you to clean up your shell history afterwards, but while
the `1p` process is running, the password may be obtained through
`/proc/PID/cmdline`. Alternatively, you may input it from the keyboard, using
option `-i`:

```sh
$ 1p -i google
Enter new password:
```

The characters you input will not show -- it's a secret!

If the service already exists in your 1Password database (as a *Login* item),
the incumbent password is replaced; otherwise a new entry is created in the
database. Other fields besides the password may be set using this command:

- Your username: `1p -u google dolphinsausage@gmail.com`
- Notes associated to the entry: `1p -n google 'Always use a VPN to check this account.\n'`
- The URL associated to the entry: `1p -U google https://gmail.com/`
- You may also rename thus the entry: `1p -T google google-dolphinsausage`

Again, option `-i` can be used in combination with the above to enter the
attribute values from standard input instead of the command line.

This is awesome, but how to fetch that information? Simply omit the setting
(last parameter) from the command:

```sh
$ 1p google-dolphinsausage
```

The password has just been copied to the system's clipboard, so you may paste
it into, say, your web browser. To echo it to standard output instead of
copying it to the clipboard, use option `-E`. Also, the options shown above to
set other fields beside the password also work for retrieval:

```sh
$ 1p -Eu google-dolphinsausage
dolphinsausage@gmail.com
$ 1p -E -n google-dolphinsausage
Always use a VPN to check this account.
```

To query the full set of information associated to an entry, you may use
option `-j`. You then get the result on standard output, in JSON format.
Do remark that it will include the password, in clear text! So
[`jq`](https://stedolan.github.io/jq/) is your friend.

```sh
$ 1p -j google-dolphinsausage | jq 'del(.password) | del(.notes)'
{
  "uuid": "ahyf6ifepbkya43fhro5hitvoy",
  "username": "dolphinsausage@gmail.com",
  "title": "google-dolphinsausage",
  "url": "https://gmail.com/",
  "vault": "Private"
}
```

To specify the vault in which to query or set a database item, append the name
of the vault to the item's name (aka it's title), separating the two with '@'.
For instance:

```sh
$ 1p -E router@Work
passw0rd_of_router!@#
```

## Storing and manipulating long documents

The *notes* field of any 1Password entry can be used to store a longer
document in your 1Password database. While notes can be queried and set
through the `-n` option as seen above, one can also open the document in their
favourite `$EDITOR` by using the `-e` option:

```sh
$ export EDITOR=vim
$ 1p -e "World domination plan"
...
```

Once the editor exits, the notes as last saved into the editor are written
back to the database.

## Listing items in the 1Password database

The simplest!

```sh
1p
```

The list shows, for each item, its UUID and the vault it lives in. Pipe in
grep to look up specific vaults, etc.

## Deleting items

Simple!

```sh
$ 1p --rm "World domination plan"   # It's not just worth it. Instead:
$ 1p -e "Universe domination plan"  # Now *that's* a good challenge.
```

## Generating strong passwords

It is **strongly suggested** to leverage the usability of a password manager
to set strong random passwords for all the services one uses. The `pwgen`
utility, installed along `1p`, can help with this. For instance, to generate a
random 30-character password for my alternate Facebook account:

```sh
$ 1p "Alternate Facebook" `pwgen 30`
```

## Option reference

Rely on good old `-h` for a complete option reference:

```sh
$ 1p -h
...  # Human, just install and try it!
```


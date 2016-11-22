# Various useful SSH related commands.

1. To specify the identity file to use when logging into a machine: 

```
ssh -i /path/to/private_key hostname
```

2. To create a new 2048 bit RSA key pair with a specific comment:

```
ssh-keygen -t rsa -C YYYYMMDD_surname_givenname
```

Example:

```
ssh-keygen -t rsa -C 20161121_anderson_ken
```

3. To copy a public key file on OS X so you can paste it into an `authorized_keys` file on a server:

```
pbcopy < ~/path/to/public_key.pub
```

Example:

```
pbcopy < ~/.ssh/id_rsa.pub
```


# Set up AWS profile

- Add AWS profile command

```
$ aws configure --profile {new_profile_name}

AWS Access Key ID [****************GIXI]: new_your_access_key
AWS Secret Access Key [****************dMYm]: new_your_secret_access_key
Default region name [ap-northeast-1]: {your_use_region}
Default output format [None]: json
```

- check AWS profile

```
$ cd ~/.aws
$ cat config

[new_profile_name]
aws_access_key_id = new_your_access_key
aws_secret_access_key = new_your_secret_access_key
```

# Use profile

```
$ export AWS_PROFILE={your_profile_name}
```

or

```
$ AWS_PROFILE={your_profile_name} npm run *
```

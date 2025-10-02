## AWS Credentials Refresh

```bash
awsref() {
  local p="${1:-default}"
  aws sso login --profile "$p" || return $?
  eval "$(aws configure export-credentials --profile "$p" --format env)"
  awk -v prof="$p" -v ak="$AWS_ACCESS_KEY_ID" -v sk="$AWS_SECRET_ACCESS_KEY" -v st="$AWS_SESSION_TOKEN" '
    BEGIN { insec=0 }
    /^\[/ {
      if ($0=="[" prof "]") { insec=1 } else { if (insec) insec=0 }
    }
    { if (!insec) print }
    END {
      print "[" prof "]"
      print "aws_access_key_id = " ak
      print "aws_secret_access_key = " sk
      print "aws_session_token = " st
    }
  ' ~/.aws/credentials > ~/.aws/credentials.new && mv ~/.aws/credentials.new ~/.aws/credentials
}
```

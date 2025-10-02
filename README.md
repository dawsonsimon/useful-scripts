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
      print "AWS_ACCESS_KEY_ID=" ak
      print "AWS_SECRET_ACCESS_KEY=" sk
      print "AWS_SESSION_TOKEN=" st
    }
  ' ~/.aws/credentials > ~/.aws/credentials.new && mv ~/.aws/credentials.new ~/.aws/credentials
}
```

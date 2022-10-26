If you are using Mac OS to generate the manifest files for Image assurance, please note the following:

Once you reached the point where you must run the 'install-ia-admission-controller.sh' script, instead of running it just read its content as:

```
curl ${URL}/install-ia-admission-controller.sh 
```

You can apply the first curl commands, and change the permissions:

```
curl ${URL}/generate-resource-yaml.sh -o generate-resource-yaml.sh
curl ${URL}/tigera-image-assurance-admission-controller.yaml -o tigera-image-assurance-admission-controller.yaml
curl ${URL}/01-tigera-secure-admission-controller-tls-secret.yaml.tpl.sh -o 01-tigera-secure-admission-controller-tls-secret.yaml.tpl.sh
curl ${URL}/06-tigera-secure-validating-webhook.yaml.tpl.sh -o 06-tigera-secure-validating-webhook.yaml.tpl.sh

chmod +x generate-resource-yaml.sh
chmod +x 01-tigera-secure-admission-controller-tls-secret.yaml.tpl.sh
chmod +x 06-tigera-secure-validating-webhook.yaml.tpl.sh
```

But once you must run the script to generate the manifest files, please edit it first and change the two 'base64 -w 0' occurrences for 'base64'
This is needed as MacOS by default does not wrap the encoded string once it reaches certain size, while linux does this by default, so the option -w is not supported or needed on MacOS
Both lines should look like:

```
export BASE64_CERTIFICATE=$(cat ${CURRENT_DIR}/admission_controller_cert.pem | base64)
export BASE64_KEY=$(cat ${CURRENT_DIR}/admission_controller_key.pem | base64)
```

Then you can continue with the following lines:

```
cp 01-tigera-secure-admission-controller-tls-secret.yaml tigera-image-assurance-admission-controller-deploy.yaml
cat tigera-image-assurance-admission-controller.yaml >>tigera-image-assurance-admission-controller-deploy.yaml
```

But for the last one, you will need to append the content of of a file into other, and add a separator between them. In the script this is done using the '-s' option of sed
This option means 'separate', however is not needed or supported on MacOS. Just remove that flag, so your command should look like:   

```
sed -e '1 s/^/---\n/g' 06-tigera-secure-validating-webhook.yaml >>tigera-image-assurance-admission-controller-deploy.yaml 
```

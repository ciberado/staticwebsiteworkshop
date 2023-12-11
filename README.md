# Aplicaciones estáticas

Este laboratorio deberías poder completarlo tanto desde CLoud Shell como con
tu workstation: las instrucciones funcionarán correctamente en Amazon Linux
y Ubuntu.

## Preparación

* El nombre del bucket forma parte de la URL asociada a cada objeto, 
por lo que no pueden existir dos con el mismo
nombre a nivel global. Define un nombre único para albergar tu blog:

```bash
BLOG_NAME=blognumber$RANDOM
```

* Muestra el nombre que se ha generado para tu bucket:

```bash
echo $BLOG_NAME
```

## Configuración de S3

* Crea el bucket con:

```bash
aws s3 mb s3://$BLOG_NAME
```

* Comprueba que el bucket está disponible con

```bash
aws s3 ls
```

* Por motivos de seguridad el acceso publico de los buckets está bloqueado por defecto.
Nosotros tenemos que eliminar dicho bloqueo:

```bash
aws s3api delete-public-access-block --bucket $BLOG_NAME
```

* Define la *resource policy* que permitirá que incluso usuarios no autenticados puedan leer de él

```bash
cat<< EOF > policy.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Public s3 access",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::$BLOG_NAME/*"
    }
  ]
}
EOF
```

* Asigna la policy al bucket. A partir de este momento cualquier persona
podrá leer de él:

```bash
aws s3api put-bucket-policy --bucket $BLOG_NAME --policy file://policy.json
```

* Genera la configuración del bucket para que se comporte como un servidor web

```bash
cat<< EOF > statichost.json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "404.html"
  },
  "RoutingRules": [
    {
      "Redirect": {
        "ReplaceKeyWith": "index.html"
      },
      "Condition": {
        "KeyPrefixEquals": "/"
      }
    }
  ]
}
EOF
```

* Asocia el bucket a dicha configuración

```
aws s3api put-bucket-website --bucket $BLOG_NAME --website-configuration file://statichost.json
```

## Publicación del blog

* Fija la región de trabajo

```bash
REGION=us-east-1
```

* Crea tu blog en la carpeta `web`

```bash
mkdir web
wget https://pastebin.com/raw/kAQg0yhu -O web/index.html
wget https://pastebin.com/raw/jxEqVD29 -O web/404.html
```

* Copia tu blog al bucket

```bash
aws s3 cp --cache-control max-age=3600 --recursive web/ s3://$BLOG_NAME
```

* Obtén la dirección pública compatible con navegadores de tu bucket y úsala
para acceder a tu blog desde el navegador:

```bash
echo The website url is: http://$BLOG_NAME.s3-website-$REGION.amazonaws.com
```



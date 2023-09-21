# Frontend deployment

We will deploy a SPA (Single Page Application) on AWS S3 with CDN (Content Delivery Network) and HTTPS enabled.

## Step 1: Create a frontend project

For this project we will use the basic [React](https://beta.es.reactjs.org/learn/start-a-new-react-project) project.

```sh
npx create-react-app ayudantia-front
npm run start
```

Now we will compile our SPA so it is client side rendered.
```sh
npm run build
```

## Step 2: Create an S3 bucket

1. Go to S3
2. Create bucket
3. Name the bucket and uncheck all `Block all public access` checkboxes.
4. Go to your bucket
5. In the `Properties` tab select `Edit` in the `Static Web Hosting`
6. Select `Enable` and change index document and error document to `index.html`
7. Go to  `Permissions` tab and `Edit` `Bucket Policy`
8. Use the following policy (replace BUCKETARN with your Bucket Arn shown in the same view)
   ```
   {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "BUCKETARN/*"
            }
        ]
    }
   ```

## Step 3: Obtain IAM credentials in AWS

This steps are not necessary if you already have your access keys.
1. Go to your Profile options in the top-right menu.
2. Select `Security credentials`
3. Select `Create access key` and create the keys.
4. Save the keys (safely).

## Step 4: Setup AWS CLI

Follow [these steps](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) to install con configure.

1. Install (only for Debian Based Systems) (If you are using Mac, use the link above)
    ```sh
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```
2. Configure using the keys created in the previous step.
    ```sh
    aws configure
    ```

## Step 5: Upload SPA or static site to S3.

1. Build your project `npm build`
2. Upload the created build to your bucket `aws s3 sync build/ s3://BUCKET --delete`
3. Now your site is uploaded, go to your AWS console, S3, your bucket, Properties tab, and copy the url of the website in the bottom of the settings.

## Step 6: Obtaining your SSL certificate

1. Change your region in your AWS console to `us-east-1`
2. Go to `Certificate Manager`
3. `Request certificate`
4. `Request a public certificate` and `Next`
5. Write all the domains that you want to use, ie. `domain.com`, `*.domain.com`.
6. Create the certificate and add the records required to your DNS.

## Step 7: Setup Content Delivery Network (AWS Cloudfront)

1. Go to `Cludfront`
2. `Create a Cloudfront distribution`.
3. For the origin, use your S3 bucket url from S3 console in properties, static web hosting.
4. In default cache behaviour, in Viewer protocol policy, select `Redirect HTTP to HTTPS`.
5. In Settings, Alternate Domain Name, write your domain name, ie. `domain.com`, `*.domain.com`.
6. In the same Settings part, Custom SSL Certificate, select the created certificate.
7. `Create distribution`

## Step 8: Setup your DNS

1. Go to your Cloudfront distribution and copy its Domain Name.
2. Go to your DNS and create a CNAME with your custom domain/domains and your cloudfront distribution domain name as its value.

## Step 9: Update your frontent content.
To update your frontend, you have to

1. Build the changes `npm build`.
2. Upload the changes to your S3 bucket `aws s3 sync build/ s3://BUCKET --delete`.
3. Invalidate the cache in Cloudfront `aws cloudfront create-invalidation --distribution-id YOUR_CLOUDFRONT_ID --paths "/*"`
4. Check if the cache was invalidated `aws cloudfront get-invalidation --id INVALIDATION_ID --distribution-id DISTRIBUTION_ID`

## Extra: Why should I use S3?
In this example, we are using S3 to deploy an SPA, static website or any static assets, but, why should we use S3?. We can, for example, use EC2 or another service and setup a webserver (such as NGINX) to serve static assets or our SPA, but S3 has some advantages such as:

1. Pricing:
   1. Máquina propia (On premise): Demanda tener un computador/servidor corriendo 24/7 y podría fallar en casos de terremotos, incendios, etc. Presenta múltiples desafíos en los hay invertir para 'solucionar'.
   2. [EC2: Se paga por](https://aws.amazon.com/ec2/pricing/on-demand/):
      1. Arriendo de capacidad de procesamiento (CPU)
      2. Ram
      3. Disco duro fijo (Si uno usa 1GB, pero arrienga 50GB por la posibilidad de llegar a esa capacidad, se paga por los 50GB).
      4. Transferencia de datos. (Cada vez que alguien ingresa a nuestro sitio)
      5. Dirección IP
      6. Otros.
   3. [S3: Se paga por](https://aws.amazon.com/s3/pricing/):
      1. Disco duro utilizado (Es flexible según demanda)
      2. Existe la opción de pagar menos por espacio de memoria que se accede con menor frecuencia.
      3. Transferencia de datos
      4. Operaciones de BD (En caso de usar S3 como bdd NoSQL)
      5. Otros
2. Throughput/Capacidad de acceso
   1. EC3:
      1. Depende de la configuración del usuario.
      2. En caso de necesitar más capacidad se debe escalar horizontal/verticalmente y configurar load balancers.
   2. S3
      1. Está optimizado para traspasar datos estáticos (sin necesitar alguna configuración propia). Con un solo S3 [aseguran soportar 5500 request por segundo](https://aws.amazon.com/about-aws/whats-new/2018/07/amazon-s3-announces-increased-request-rate-performance/)
      2. En caso de necesitas más capacidad, se puede utilizar [Cloudfront](https://aws.amazon.com/cloudfront/pricing/) como Content Delivery Network.
3. Ease of use
4. More info [Here](https://aws.amazon.com/s3/)

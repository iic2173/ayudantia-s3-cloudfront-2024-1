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

1. Install
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

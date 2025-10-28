# CLOUD *** AWS Enumeration 

jueves, 28 de noviembre de $2024 \quad 12: 24$

Intro: to correctly identify valuable insights we need to have clear objectives and understand what we are trying to achieve. Are we searching for vulnerabilities, business insights or system inefficiencies? Do we need to escalate privileges to obtain more insights? Having a clear goal helps us to focus on relevant data.

Point de départ= DNS. On regarde qui sont les serveurs DNS qui contiennent les enregistrements pour ce domaine : host -t ns offseclab.io
Puis quand on a le serveur en question, on peut approfondir: whois awsdns-00.com | grep "Registrant Organization"
Puisqu'il s'agit ici d'Amazon, important de savoir que le service "Route 53" est utilisé (https://aws.amazon.com/route53/).

Puis on cherche simplement l'adresse IP du domaine avec une simple commande "host" par exemple.
kali@kali: $ host 52.70.117.69
69.117.70.52.in-addr.arpa domain name pointer ec2-52-70-117-69.compute-1.amazonaws.com
kali@kali: $ whois 52.70.117.69 | grep "OrgName"
OrgName: Amazon Technologies Inc.
With the whois lookup, we realize that the public IP belongs to Amazon and with the reverse lookup, we learn two things: it's a resource hosted in AWS (amazonaws.com) and the resource is an Amazon Elastic Compute Cloud (Amazon EC2) instance. Note: The EC2 instance is a virtual machine in the AWS cloud. EC2 is a common service used to host websites, applications, and other services that require a server.

Puis, on passe par la phase de reco passive & OSINT. C'est une phase récursive. C'est-à-dire que quand on trouve quelque chose (subdomain ou @ IP publique par ex), on recommenance l'exploration.
Cette phase n'a pas été faite dans le lab offsec car il ne s'agit pas d'un vrai domaine. Voir peut-être TCM qui je crois appliquais cette phase à Tesla.

Puis, on cherche les domaines avec par ex dnsenum: dnsenum offseclab.io --threads 100
Ensuite grace au web developper tool (onglet network) on regarde les GET pour identifier d'éventuels buckets S3 (ou d'un autre service de stockage si ce n'est pas Amazon). Ex: from the path offseclab-assets-public-
axevtewi/sites/www/images/amethyst.png, we can learn that the S3 bucket name is offseclab-assets-public-axevtewi and the object key is sites/www/images/ruby-expanded.png. Voir si on arrive à explorer ce bucket grâce à son URL ou si on a un Access Denied.

Ici axevtewi est un identifiant unique de bucket (qui a l'origine devait prévenir les énumération mais bon pas top comme protection...). On peut par exemple essayer d'énumérer les noms de bucket, comme par exemple avec offseclab-assets-private-axevtewi.

Outils pour ce genre d'énumération: cloudbrute or cloud-enum (cli = cloud_enum). The cloud-enum tool will search through several public CSPs for resources containing a keyword specified using the --keyword KEYWORD (-k KEYWORD) parameter. We can specify multiple keyword arguments, or we can specify a list with the --keyfile KEYFILE (-kf KEYFILE) parameter.

Exemples:

- cloud_enum -k offseclab-assets-public-axevtewi --quickscan --disable-azure --disable-gcp
- for key in "public" "private" "dev" "prod" "development" "production"; do echo "offseclab-assets-$key-axevtewi"; done / tee /tmp/keyfile.txt puis cloud_enum -kf/tmp/keyfile.txt -qs --disable-azure --disable-gcp

| AWS | Azure | GCP |
| :-- | :-- | :-- |
| s3.amazonaws.com | web.core.windows.net | appspot.com |
| awsapps.com | file.core.windows.net | storage.googleapis.com |
|  | blob.core.windows.net |  |
|  | azurewebsites.net |  |
|  | cloudapp.net |  |
| Table 1 - Custom URLs of All The Three Major CSPs |  |  |

Reco via API:

One way is via a web application that acts as a portal for cloud services provided by the CSP. Access is protected by credentials (username, password, MFA, etc).
Another way is through APIs that allow customers to interact programmatically, integrating with custom solutions and even other cloud platforms. The API is publicly available, but requires authentication to interact with it.

Culture gé: parmi les ressources offertes par Amazon et qu'on va chercher à découvrir:

Publicly-shared Amazon Machine Images (AMIs) (dans la logique Openstack, les "images" je crois)
Publicly-shared Elastic Block Storage (EBS) snapshots
Relational Databases (RDS) snapshots

Le truc c'est que ces ressources n'ont pas de nom de DNS ou autre pour y accéder publiquement, donc on va essayer de les débusquer via l'API.

Le principe c'est de trouver des images (si on prend l'AMI comme ex) via des filtres et tomber sur des accounts ID qu'on énumère ensuite:
aws --profile attacker ec2 describe-images --executable-users all --filters "Name=name,Values=*Offseclab*"
Avec des snapshots EBS:
aws --profile attacker ec2 describe-snapshots --filters "Name=description,Values=*offseclab*"
Si pas de ressources dispo publiquement, on ne trouvera pas d'account Id. À l'aide d'un compte qui a un bucket S3, on va s'y prendre autrement, toujours grâce à l'API : we'll begin by creating an IAM user that, by default, won't have any permissions to execute actions. Then we'll add a policy to grant read access to the bucket with the Condition that the permission will only apply if the account ID that owns the bucket starts with the digit " $x$ ". If we can't read the bucket, we'll keep trying with other numbers until we are able to read the bucket, showing we've identified the first digit of the account ID where the bucket resides. We can iterate through the other digits until we retrieve all the account IDs

First, we'll choose a publicly-readable bucket or object inside the target account. Because the bucket/object is publiclyreadable, we should be able to list the content of it with any IAM user of any AWS account. In the lab, we'll choose one of the publicly-readable buckets.

Then, we'll create a new IAM user in our attacker account. By default, IAM users don't have any permissions to execute any actions, so the new user won't be able to list the content of the public resource even when it's public.

Next, we'll create a policy that will grant permissions to list buckets and read objects. However, we'll add the Condition that the read permission will only apply if the account ID that owns the bucket starts with the digit " $x$ ".

After we apply the policy to the new IAM user, we'll test if we can list the bucket with the new user's credentials. We'll
test the value $x$ from 0 to 9 until we can list the bucket, meaning that we found the first digit of the account.
D'autres technique d'enum via API son possibles mais je fatigue..Voir outil "pacu" qui permet d'énumrer et de sortir des infos sur les roles et les utilisateurs.

Quelques commandes utilisées lors des lab:
aws configure --profile attacker
aws --profile attacker s3 mb s3://offseclab-dummy-bucket-$RANDOM-$RANDOM-$RANDOM
aws configure --profile enum
aws sts get-caller-identity --profile enum // pour avoir l'account ID
nano grant-s3-bucket-read.json
aws --profile attacker iam create-user --user-name enum // cette dommande donne l'identity id qui est très important:
- $ aws sts get-caller-identity --profile enum
{
"UserId": "AIDARCLNMVYYWNKKUWRLB",
"Account": "073780735537",
"Arn": "arn:aws:iam::073780735537:user/attacker"
}
$ aws --profile attacker iam create-user --user-name enum
{
"User": {
"Path": "/",
"UserName": "enum",
"UserId": "AIDARCLNMVYYVZ4S7K67A",
"Arn": "arn:aws:iam::073780735537:user/enum",
"CreateDate": "2024-12-11T11:16:34+00:00"
}
}
Exemple de politique pour permettre l'accès aux ressources (ici l'action d'énumérer les VPC).
$ more policy.json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "ec2:DescribeVpcs",
"Resource": "arn:aws:iam::073780735537:role/amethyst-lab_admin"
}
]
}
The get-caller-identity subcommand is a good way to identify the account and identity of the credentials, and this action will never return an AccessDenied error. However, we should be aware that this action is logged in Cloudtrail's
event history. As defenders, we should establish alerts for these types of calls as they are typically executed by attackers once they've compromised credentials.

Finally, let's suppose that our compromised credentials don't have the privileges to query for IAM-related information. In this case, we need to adopt a brute-force approach, meaning we'll run several actions on services in hopes of finding one that doesn't produce an authorization error.

We can build a script to automate this brute-force approach or leverage popular AWS tools such as pacu (Module: iam__bruteforce_permissions), awsenum or enumerate-iam. We can use any of these approaches to discover permissions in compromised credentials.

A propos de politiques de sécurité, mappées à des utilisateurs ou groupes:
From the output, we learn that the group doesn't have any inline policy, but it does have an attached managed policy. This particular policy is classified as an AWS Managed Policy, a special set of policies provided by AWS with pre-defined permissions to quickly attach to IAM Identities. However, a word of caution: these policies often grant broader permissions than might be desired. Ideally, they should be paired with other more-restrictive policies to ensure finegrained permission control.

Warning: While AWS Managed Policies offer flexibility for IAM management, they tend to be overly-permissive and there is an inherent security risk when they are used alone.

We can filter the results using the --filter and a list of space-separated elements which may include User, Role, Group, LocalManagedPolicy, and AWSManagedPolicy. We are not interested in AWS custom-managed policies so we'll omit this value:
aws --profile target iam get-account-authorization-details --filter User Group LocalManagedPolicy Role | tee account-authorization-details.json
Note: cette commande est bien car elle renvoie pas mal d'information et génére moins de logs que si on avait lancé plusieurs commandes à la suite pour avoir les usersn puis les roles etc...

Sans transition, une petite utilisation de l'utilisation du filtre --filter (donc le filtre se fait côté serveur) : aws --profile target iam get-account-authorization-details --filter User --query "UserDetailList[].UserName"

Même chose mais on sélectionne plusieurs keys:
aws --profile target iam get-account-authorization-details --filter User --query "UserDetailList[0].\{Name: UserName,Path: Path,Groups: GroupList\}"

Mais juste pour le premier tuple.
Maintenant on veut matcher par exemple les user dont le nom contient "admin", mais avec --query (donc côté client, avec du JMESPath):
aws --profile target iam get-account-authorization-details --filter User --query "UserDetailList[?contains(UserName, 'admin')].\{Name: UserName\}"

As a final example, we'll build an expression that selects elements from different objects. In this case, let's gather all the names of the IAM Users and Groups which contain "/admin/" in their Path key.
aws --profile target iam get-account-authorization-details --filter User Group --query "\{Users: UserDetailList[? Path=='/admin/'].UserName, Groups: GroupDetailList[?Path=='/admin/'].\{Name: GroupName\}\}"
Ressources:
https://jmespath.org/examples.html
https://docs.aws.amazon.com/cli/v1/userguide/cli-usage-filter.html
In summary, the --query parameter is a great AWS CLI feature that leverages JMESPath expressions. Azure CLI also
implements a similar feature with JMESPath expressions. Other CSPs like Google Cloud don't implement this directly but they support displaying output in JSON so we can use external tools to process the data.

Attribute-Based Access Control (ABAC) is an authorization strategy in which a subject's permission to perform a set of operations is determined by evaluating attributes associated with that subject. Tags are commonly used as attributes to implement ABAC in public cloud environments.

But in the real world, sifting through the potentially large volume of enumeration data in search of actionable insights can be highly labor-intensive.
Automated tools can help manage this data. They are typically designed to present the data in a more organized and visual manner, making it easier to understand the details. For example, Cloudmapper provides visual representations of AWS configurations, helping identify potential issues. We also mentioned Awspx which is useful for analyzing how IAM resources interlink. The selection of a particular tool often depends on the type of assessment and the desired outcomes.

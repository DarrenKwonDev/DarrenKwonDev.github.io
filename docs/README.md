# darrenkwondev.github.io

## hugo

```bash
hugo new site .
git init
git submodule add $theme_git_path themes/$theme_name
echo "theme = '$theme_name'" >> hugo.toml
hugo server

hugo hugo new content posts/$post_name.md
```

[configuration](https://gohugo.io/getting-started/configuration/)  
[deploy](https://gohugo.io/hosting-and-deployment/hugo-deploy/)  
hugo version : hugo v0.118.2-da7983ac4b94d97d776d7c2405040de97e95c03d+extended darwin/arm64 BuildDate=2023-08-31T11:23:51Z VendorInfo=brew

## theme

[docs](https://rxsamira.netlify.app/post/hugo-dead-simple/)
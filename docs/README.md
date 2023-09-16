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

## theme

[docs](https://rxsamira.netlify.app/post/hugo-dead-simple/)

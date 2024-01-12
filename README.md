## Developer Portfolio Landing Page Template

Use this template if you need a quick developer / data science portfolio! Based on a Minimal Jekyll theme for GitHub Pages.

See full step by step tutorial [on Medium](https://medium.com/@evanca/set-up-your-portfolio-website-in-less-than-10-minutes-with-github-pages-d0efa8ff56fd).
___

You can use the editor on GitHub to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```
For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

## Building Jekyll
```shell script
bundle install
```

## Running Jekyll
```shell script
bundle exec jekyll serve
```

## Building Docker
```shell script
docker build -t jekyll-app:1.0.0 .
```

## Deployment yaml
```shell script
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jekyll-website
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jekyll-website
  template:
    metadata:
      labels:
        app: jekyll-website  
    spec:
      containers:
      - name: website
        image: jekyll-app:1.0.0
        ports:
          - containerPort: 80
```

___



References:

[1] Jekyll theme "Minimal" for GitHub Pages: https://github.com/pages-themes/minimal (CC0 1.0 Universal License)
[2] https://www.cloudytuts.com/guides/kubernetes/how-to-deploy-jekyll-on-kubernetes/

---

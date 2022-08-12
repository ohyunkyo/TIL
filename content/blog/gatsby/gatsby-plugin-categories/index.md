---
title: "개츠비 카테고리 플러그인 사용하기"
date: "2022-08-02"
last_modified_at: "2022-08-00"
category: gatsby
---

## 0. 서론
내가 만든 [블로그](https://ohyunkyo.github.io) 와 [TIL](https://ohyunkyo.github.io/TIL) 는 일반적인 블로그 양식과 달리, 카테고리별로 게시물들이 분류되어 있지 않고 전체 게시물 목록만을 출력한다. 
그래서 원하는 게시글을 찾기 어렵고 어떤 주제에 관한 글이 있는지 직관적으로 판단하기 어렵다.

이것을 각각의 카테고리 별로 목록 페이지를 생성해 해당 카테고리의 게시물만을 보여주도록 수정한 다음, 이번에 기획중인 포트폴리오의 profile 부분에 블로그 링크들을 추가하면 좋을것 같았다.

## 1. 카테고리 사용 계획
내가 만들고자 하는것은 다음과 같다.

### 1.1 각 카테고리 별 목록 페이지 추가
- `/category/{category_name}` 페이지가 있어야 한다.
- 해당 페이지에서 각 카테고리에 해당하는 게시물 목록을 출력한다.
### 1.2 모든 페이지에서 카테고리 메뉴 바 출력하기
- 각 카테고리 별 목록 페이지로 이동 가능한 메뉴 바를 만들고 모든 페이지에서 볼수 있도록 한다.
### 1.3 각 게시물 페이지에서 해당 카테고리 목록 페이지로 이동
- 게시물 페이지에 해당 카테고리의 목록 페이지로 이동 가능한 링크 출력

## 2. 기능 구현할 방법 찾아보기
위 계획을 구현하기 위해 여러 방법을 찾아봤다. 많은 사람들이 다양한 방법으로 구현한 예제가 있었지만 리액트를 전혀 모르는 나에겐 적용하기 어려웠다.  
그때 간단히 사용 가능한 플러그인을 발견했다.

## 3. 예제 적용하기
내가 만든 블로그에 플러그인 예제를 적용하는 방법을 정리한다.

> 블로그는 [github pages 를 사용하여 블로그 만들기](https://ohyunkyo.github.io/git/github-page-make-blog/) 와 같은 방법으로 만들었다.

## 3.1 category 추가
가장 먼저 해야할 것은 `*.md` 파일들에 `category` 라는 `front-matter` 를 추가하는것이다. 이 마크다운 문서는 아래와 같은 `front-matter` 를 가지고 있는데 다른 `markdown` 문서에도 `category` 를 추가해주면 된다.

```markdown
---
title: "개츠비 카테고리 플러그인 사용하기"
date: "2022-08-02"
last_modified_at: "2022-08-00"
category: react
---
```

## 3.2 설치
다음으로는 해당 플러그인을 설치한다. `README.md` 에서는 `yarn` 을 사용했지만 나는 `npm` 을 사용한다.

```shell
$ npm install gatsby-plugin-categories 
```

## 3.3 파일 추가
예제 적용을 위한 파일을 추가한다. 

```javascript
# src/components/PostsList.js

import React from "react"
import PostsListCard from "./PostsListCard"

const PostsList = ({ postEdges }) => {
  return postEdges.map(({ node }) => {
    return <PostsListCard key={node.fields.slug} {...node} />
  })
}

export default PostsList
```

```javascript
# src/components/PostsListCard.js

import React from "react"
import { Link } from "gatsby"
import { Card } from "react-bootstrap"

const PostsListCard = ({ frontmatter, fields, excerpt }) => {
  const title = frontmatter.title || fields.slug

  return (
    <Card className="mb-4">
      <Card.Body>
        <h2 className="card-title">{title}</h2>
        <div
          dangerouslySetInnerHTML={{
            __html: frontmatter.description || excerpt,
          }}
        />
        <Link to={`/${fields.slug}/`} className="btn btn-primary">
          Read More &rarr;
        </Link>
      </Card.Body>
      <Card.Footer className="text-muted">
        Posted on {frontmatter.date}
      </Card.Footer>
    </Card>
  )
}

export default PostsListCard
```

```javascript
# src/templates/category.js

import React from "react"
import { graphql } from "gatsby"
import { Container } from "react-bootstrap"

import Layout from "../components/layout"
import SEO from "../components/seo"
import PostsList from "../components/PostsList"

const CategoryTemplate = ({ location, pageContext, data }) => {
  const { category } = pageContext
  return (
    <Layout location={location} title={`Posts in category "${category}"`}>
      <div className="category-container">
        <SEO title={`Posts in category "${category}"`} />

        <Container>
          <h1>Category: {category}</h1>
          <PostsList postEdges={data.allMarkdownRemark.edges} />
        </Container>
      </div>
    </Layout>
  )
}

export const pageQuery = graphql`
  query CategoryPage($category: String) {
    allMarkdownRemark(
      limit: 1000
      filter: { fields: { category: { eq: $category } } }
    ) {
      totalCount
      edges {
        node {
          fields {
            slug
            category
          }
          excerpt
          timeToRead
          frontmatter {
            title
            date
          }
        }
      }
    }
  }
`

export default CategoryTemplate
```

## 3.4 파일 수정
다음으로는 플러그인을 불러오고 내가 원하는 기능을 사용하기 위해 기존 파일을 수정한다.

```javascript
# gatsby-config.js

plugins: [
...
  {
    resolve: "gatsby-plugin-categories",
    options: {
      templatePath: `${__dirname}/src/templates/category.js`,
    },
  },
]
```

```javascript
# src/templates/blog-post.js
...
import { Container, Row, Col, Card } from "react-bootstrap"
...

# 적당한 위치에 추가
<Col md={4}>
  {post.fields.category && (
    <Card className="my-4">
      <Card.Header>Filed Under</Card.Header>
      <Card.Body>
        <Link to={`/category/${post.fields.category}`}>
          {post.fields.category}
        </Link>
      </Card.Body>
    </Card>
  )}
</Col>

...

export const pageQuery = graphql`
  query BlogPostBySlug(
    $id: String!
    $previousPostId: String
    $nextPostId: String
  ) {
    site {
      siteMetadata {
        title
      }
    }
    markdownRemark(id: { eq: $id }) {
      id
      excerpt(pruneLength: 160)
      html
      frontmatter {
        title
        date(formatString: "MMMM DD, YYYY")
        description
      }
      /* 추가한 부분 */
      fields {
        category
        slug
      }
      /*-----------*/
    }
    previous: markdownRemark(id: { eq: $previousPostId }) {
      fields {
        slug
      }
      frontmatter {
        title
      }
    }
    next: markdownRemark(id: { eq: $nextPostId }) {
      fields {
        slug
      }
      frontmatter {
        title
      }
    }
  }
`
```

> 만약 설치되지 않은 npm 패키지가 있다면 따로 설치해준다.



## 99. 끝나고 나서
개인적으로 `gatsby-plugin-categories` 의 `README.md` 파일의 설명이 굉장히 부족하다고 느꼈다.  
다른 참조할만한 문서도 없어서 소스코드만을 보고 구현에 성공했는데, 어떤 파일이 필요하고 각 파일이 어떤 역할을 하는지에 대해서는 기본적으로 제공해야 한다고 생각한다.

내가 앞으로 어떤식으로든 소프트웨어를 만들게 되면 최대한 자세한 설명을 포함하는 문서를 제공해야겠다는 생각을 했다.

## References

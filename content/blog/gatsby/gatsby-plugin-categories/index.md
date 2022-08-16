---
title: "개츠비 카테고리 플러그인 사용하기"
date: "2022-08-02"
last_modified_at: "2022-08-12"
category: gatsby
---

## 0. 서론
내가 `gatsby` 를 사용해 만든 [블로그](https://ohyunkyo.github.io) 와 [TIL](https://ohyunkyo.github.io/TIL) 는 일반적인 블로그 양식과 달리, 게시물들이 분류되어 있지 않고 전체 게시물 목록만을 출력한다. 
그래서 원하는 게시글을 찾거나 어떤 주제에 관한 글이 있는지 직관적으로 판단하기 어렵다.

이 문제를 해결하기 위해 여러 방법을 찾아봤다. 많은 사람들이 다양한 방법으로 구현한 예제가 있었지만 리액트를 전혀 모르는 나에겐 적용하기 어려웠다.  
그때 간단히 사용 가능한 `gatsby_plugin_categories` 라는 플러그인을 발견했는데, 이 플러그인을 사용해 게시물을 분류하는 방법을 정리한다.

### 0.1. 카테고리 사용 계획
내가 필요한 것은 다음과 같다.

#### 0.1.1 각 카테고리 별 목록 페이지
- `/category/{category_name}` 페이지가 있어야 한다.
- 해당 페이지에서 각 카테고리에 해당하는 게시물 목록을 출력한다.
#### 0.1.2 각 카테고리 별 목록 페이지로 이동 가능한 네비게이션 메뉴
- 각 카테고리 별 목록 페이지로 이동 가능한 네비게이션을 만들고 모든 페이지에서 볼수 있도록 한다.
#### 0.1.3 각 게시물 페이지에서 해당 카테고리 목록 페이지로 이동
- 게시물 페이지에 해당 카테고리의 목록 페이지로 이동 가능한 링크 출력

> 블로그는 [github pages 를 사용하여 블로그 만들기](https://ohyunkyo.github.io/git/github-page-make-blog/) 와 같은 방법으로 만들었다.

## 1. 마크다운 파일 수정하기
가장 먼저 해야할 것은 `*.md` 파일들에 `category` 라는 `front-matter` 를 추가하는것이다.   
이 마크다운 문서는 아래와 같은 `front-matter` 를 가지고 있는데, 이것처럼 `category` 를 추가하면 된다.

```markdown
---
title: "개츠비 카테고리 플러그인 사용하기"
date: "2022-08-02"
last_modified_at: "2022-08-00"
category: react
---
```

## 2. 각 카테고리 별 목록 페이지 생성
이제 방금 추가한 `category` 별로 목록 페이지를 생성하고 해당 목록 페이지에서 각 카테고리의 게시물 목록을 출력하도록 할것이다.  
이 부분은 `gatsby_plugin_categories` 의 예제 소스를 가져와서 적용하기만 하면 쉽게 구현 가능하다.

### 2.1 설치
먼저 해당 플러그인을 설치한다. `README.md` 에서는 `yarn` 을 사용하는 방법을 안내하지만 나는 `npm` 을 사용했다.

```shell
$ npm install gatsby-plugin-categories 
```

### 2.2 파일 수정
플러그인을 불러오도록 `gatsby-config.js` 파일을 수정한다.

```javascript
// gatsby-config.js

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

`gatsby-plugin-categories` 는 `graphql query` 로 전체 카테고리 목록을 가져온다. 그 뒤 `set` 객체에 담아 중복을 제거하고 `templatePath` 옵션의 템플릿을 사용하여 각 카테고리별 목록 페이지를 생성하게 된다.


### 2.3 파일 추가
다음으로는 예제 파일을 참고하여 플러그인 사용을 위한 파일을 추가한다. 

#### 2.3.1 category
가장 먼저 `category` 템플릿 파일을 추가했다.

```javascript
// src/templates/category.js

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

#### 2.3.2 PostsList
그 다음으로는 `category` 템플릿에서 사용하는 `PostsList` 컴포넌트를 추가한다.  
여기에서는 `category` 로부터 해당 카테고리의 게시물 목록을 전달받아 출력한다.

```javascript
// src/components/PostsList.js

import React from "react"
import PostsListCard from "./PostsListCard"

const PostsList = ({ postEdges }) => {
  return postEdges.map(({ node }) => {
    return <PostsListCard key={node.fields.slug} {...node} />
  })
}

export default PostsList
```

#### 2.3.3 PostsListCard
`PostsList` 가 게시물 목록을 출력할 때에 개별 게시물 정보를 `PostsListCard` 컴포넌트에 전달한다.  
`PostsListCard` 는 하나의 게시물 정보를 받아 목록의 요소로 사용될 수 있도록 가공 후 리턴한다.      
나는 `PostsListCard` 에서 리턴하는 데이터가 `index` 페이지에서 출력되는 목록의 요소와 동일하게 보이도록 수정했다.

```javascript
// src/components/PostsListCard.js

import React from "react"
import { Link } from "gatsby"

const PostsListCard = ({ frontmatter, fields, excerpt }) => {
  const title = frontmatter.title || fields.slug

  return (
    <ol style={{ listStyle: `none` }}>
      <li key={fields.slug}>
        <article
          className="post-list-item"
          itemScope
          itemType="http://schema.org/Article"
        >
          <header>
            <h2>
              <Link to={fields.slug} itemProp="url">
                <span itemProp="headline">{title}</span>
              </Link>
            </h2>
            <small>{frontmatter.date}</small>
          </header>
          <section>
            <p
              dangerouslySetInnerHTML={{
                __html: frontmatter.description || excerpt,
              }}
              itemProp="description"
            />
          </section>
        </article>
      </li>
    </ol>
  )
}

export default PostsListCard
```

> 만약 설치되지 않은 npm 패키지가 있다면 따로 설치해준다.

## 3. 각 카테고리 별 목록 페이지로 이동 가능한 네비게이션 메뉴 생성
다음으로는 생성된 카테고리 별 목록 페이지로 이동 가능한 링크가 있는 네비게이션 메뉴를 생성해야 한다.  
일단 `components` 디렉토리에 `NavBar` 라는 컴포넌트를 생성했다.
```javascript
// src/components/NavBar.js

import * as React from "react"
import { Link, graphql, useStaticQuery} from "gatsby"

const NavBar = () => {
  const { allMarkdownRemark } = useStaticQuery(
    graphql`
      query {
        allMarkdownRemark {
          nodes {
            fields {
              category
            }
          }
        }
      }
    `
  )

  const categorySet = new Set();

  allMarkdownRemark.nodes.forEach(({
   fields
  }) => {
    if (fields && fields.category) {
      categorySet.add(fields.category);
    }
  });

  const categoryArr = Array.from(categorySet)

  return (
    <div>
      카테고리 : &nbsp;&nbsp;&nbsp;&nbsp;
      {categoryArr.map(category => {
        return <span><Link to={`/category/${category}`}>{category}</Link>&nbsp;&nbsp;&nbsp;&nbsp;</span>
      })}
      <br/><br/><br/>
    </div>
  )

}

export default NavBar
```

`gatsby-plugin-categories` 가 각 카테고리 별 목록 페이지를 생성하는 부분을 참고하여 카테고리 별 목록 페이지로 이동 가능한 링크를 만들었다.
이후 필요한 페이지에서 컴포넌트를 호출해 사용한다.

```javascript
// src/pages/index.js

...
  return (
    <Layout location={location} title={siteTitle}>
      <Seo title="All posts" />
      /* 추가한 부분 */
      <NavBar />
      /*------------*/
      <Bio />
      <ol style={{ listStyle: `none` }}>
...
```
## 4. 각 게시물 페이지에서 해당 카테고리 목록 페이지로 이동
마지막으로 각 게시물 페이지에서 해당 게시물의 카테고리 목록 페이지로 이동 가능한 버튼을 추가한다.  
이 기능은 예제의 소스코드 그대로 사용했다.
```javascript
// src/templates/blog-post.js
...
import { Container, Row, Col, Card } from "react-bootstrap"
...

// 적당한 위치에 추가
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
      /*------------*/
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

## 99. 끝나고 나서
개인적으로 `gatsby-plugin-categories` 의 `README.md` 파일의 설명이 굉장히 부족하다고 느꼈다.  
다른 참조할만한 문서도 없어서 소스코드만을 보고 구현에 성공했는데, 어떤 파일이 필요하고 각 파일이 어떤 역할을 하는지에 대해서는 기본적으로 제공해야 한다고 생각한다.

내가 앞으로 어떤식으로든 소프트웨어를 만들게 되면 최대한 자세한 설명을 포함하는 문서를 제공해야겠다는 생각을 했다.

> [2022-08-16] 개츠비의 구조에 대해 조금 알게된 지금 생각해보니 소스코드 만으로도 어느정도 이해 가능하다는 생각이 든다. 하지만 자신이 만든 프로그램에 대한 설명은 자세할수록 좋다는 생각은 변함이 없다. 

## References
[gatsby-plugin-categories](https://github.com/rmcfadzean/gatsby-pantry/tree/master/packages/gatsby-plugin-categories)  
[예제](https://github.com/rmcfadzean/gatsby-pantry/tree/master/examples/starter-blog#readme)
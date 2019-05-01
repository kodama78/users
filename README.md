## Description
#### This project specifically taught me how to initiate graphql queries and mutations on the backend inside a express server

## Technologies Used
  - **graphql** - query language that is back-end agnostic. Is meant to work with different databases and backend languages
  - **axios** - AJAX library
  - [**json-server**](https://github.com/typicode/json-server) - used to create a simple localhost server. Great for creating mock server that you need to make requests to  
  - [**express-graphql**](https://github.com/graphql/express-graphql) - module that creates a GraphQL HTTP server with Express
  - **GraphiQL** - browser tool that allows you to test queries and mutations (on root/graphql)
  
## Learnings
###Restful APIs vs. GraphQL

**Rest**
- Restful tends to start breaking down when you get to large amounts of nested data
- If your data is separated into multiple relational databases, the number of requests can grow quite large
- Restful have a tendency to overfetch or underfetch data
- To prevent this overfetching and underfetching, custom endpoints are created, which then tend to break restful conventions

**GraphQL**
- GraphQL looks at your data as interconnected graphs
- Uses queries to fetch the exact data you need in the request
- Uses mutations to create/edit data in the request

###GraphQL w/ Express
- Express handles the HTTP requests and response
- GraphQL is designated routes by Express `.use` that point it to the GraphQL module:
```
    const expressGraphQL = require('express-graphql');

    app.use('/graphql', expressGraphQL({
        schema,
        graphiql:true
    }));
```
- You identify the route that you want the module to identify it as a graphQL route, and then you provide the schema,
and in this case the ability to turn the graphiql browser tool on

###Schemas
GraphQL requires schemas to work. They tell your GraphQL application exactly what your data looks like, specifically what each 
object looks like and how they relate to each other

This is where you import the graphql library:
`const graphql = require('graphql');`

And pull off the methods that you'll be using to define your schema:
```
const {
       GraphQLObjectType,
       GraphQLString,
       GraphQLInt,
       GraphQLSchema,
       GraphQLList,
       GraphQLNonNull
   } = graphql;
```

The first major item that is needed in a schema is a RootQuery. This is considered the **entry point to your schema**.
It requires the **name** and **fields** parameters as laid out below
```angular2html
    const RootQuery = new GraphQLObjectType({
       name: 'RootQueryType',
       fields: {
          user: {
            type: UserType,
            args: { id: { type: GraphQLString } },
            resolve(parentValue, args) {
              return axios.get(`http://localhost:3000/users/${args.id}`)
                .then(resp => resp.data);
            }
          },
          company: {
            type: CompanyType,
            args: { id: { type:GraphQLString } },
            resolve(parentValue, args) {
    	        return axios.get(`http://localhost:3000/companies/${args.id}`)
                .then(resp => resp.data);
            }
          }
        }
    });
```
- In the `fields` of the RootQuery, we identify the parameters that a front end developer can query, in this case
`user` and `company`. It is best to think of these as the "entry points" into our GraphQL schema.
- If the FED queries for `user` they will hit the `type: UserType` which identifies what `fields` it can return
within it's own declaration. 
- The `args` is the arguments needed when querying for this particular field, in this case the `id`.
- The `resolve` function is the route we use to retrieve that particular information, in this case the `user` identified by `id`.
    - It requires the `parentValue` and `args` arguments
        - **parentValue** - not really used in the RootQuery, but is used in the other Types
        - **args** - the arguments from above
        
This is passed to the GraphQLSchema in the query parameter:
```angular2html
module.exports = new GraphQLSchema({
  query: RootQuery,
  mutation
});
```

The second major item to define in the schema file is the Types:
```angular2html
const CompanyType = new GraphQLObjectType({
  name: 'Company',
  fields: () => ({
	  id: { type: GraphQLString },
	  name: { type: GraphQLString },
	  description: {type: GraphQLString },
      users: {
	    type: new GraphQLList(UserType),
      resolve(parentValue, args) {
	      return axios.get(`http://localhost:3000/companies/${parentValue.id}/users`)
		      .then(resp => resp.data);
	    }
    }
  })
});
```
The method `GraphQLObjectType` instantiates a new object for graphql to consume and tells it what it looks like.
There are two required parameters in the object: 
- **name** - typically upper case and describes the type we're defining
- **fields** - these are the properties that the new Type will have
    - each fields' type needs to be defined using a GraphQL method from above
    - fields is what allows GraphQL to jump 
- **users** - you'll see that this field has a `type` and then a `resolve()` function. This is how you access other Types
within other queries. 
    - **type** - identify which Type you are trying to access
    - **resolve** - works the same. You pass the same parameters except you make use of the `parentValue` argument.
    In this case you'll get a `companyId`, which comes from the user inside the database. You then run query using that
    `companyId` to fetch the information on the company
    

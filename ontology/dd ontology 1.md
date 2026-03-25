I'm a senior developer from docdigitizer.com, an state of the art of IDP.

We have a common problem, we need to extract information from files and fit those extractions into a json schema. The problem is that if we extract a document in an open loop with LLM what will happen is that for every extraction we may get a different json schema. Thus we need to create a schema engine that allow us to feed to the extractor the right schema.

We have 3 scenarios:
1 - Caos - when we do not know anything about the document that is being sent
2 - Semi Caos - when we have some boundaries of the document types
3 - Organized - when our customers can define a list of target schemas.

Now, we want to develop a solution that can actually tackle all scenarios with a single elegant solution. This solution must be both fast, accurate and precise.

We imagine a possible solution, but feel free to: 1) suggest alternative solutions 2) benchmark across best of breed solution. Keep in mind that we are solving a very specific soluition. We are not selling a complete ontology solution. We dont want to go the univsersity, R&D route.

What we are thinking is:
a) Having a database of schemas indexed by Id
b) We can have bags of schemas, lets say schema groups. A schema group is nothing more that a group of schemas
c) An organization Id can be linkedin to one or more schema group. 
d) When I send to extract a document, the organization can send also the schema group ID

Now the problem is the following. In the real world we have to have a category identification mechanism so that I first identify what is the category of the document, and then execute the extraction. This category extraction step (CE) is critical. Why? Because asking the LLM to identifiy a document out of 1,000,000 possible schemas will exploded tokens, time and precision.

Thus we need a better way to do this. One possible way is to do this a an iteration cicle where we go top down , we first ask the system what level 1 category the document is (which for example may have only 20 categories). Then after 1st cycle , you can iterate for level 2. Easy to say that this will support polinomial size of categories. We believe at the 3 or 4th level at most we can greatly pinpoint a sufficeint small number of potencial schemas so that we can feed those to the step 2: the extraction step.

The beatuy of this is this "special" categorization documents can also be schemas themselves. It's easy to see that a categorization level is nothing more than running an extraction data from a document where the schema forces the extraction to bind to a special structure where we can have only 1 field: the level1 category

Continuing to go on thi s route , every other schema can be tagged with the nLevel category levels. That means that after the CE special step we are now able to execute the extraction with the schemas that comply with the category restrictions.

Now, for the caos approach , we will want the system to extract but it is possible that we thinks that isnot a sufficent close match to a schema, at this point the system will be able to suggest a new schema. This means that as documents inbound we will have a long tail of schemas. Thus we need to maintain some order in the system. On way is to create these new schemas with a maturity flag: say "community", while all the other schemas validated are in "production" level. Of course this will evolve across time and categories and schemas will be evolving. Thus, the special schemas for CEmust be versioned, as all schemas in the system will be also. We will be maintaining all schemas , as schema "space" ocupied nowadays is pretty cheap.

So a document arrives, it executes a CE step iterating levels of categories until it reaches the point where there is no more sub categories. He then run the extraction using the schemas resulted.

In the 3rd scenario, the orgnaized already specified a schema group ID, but in real world that group could have hundreds of schemas, and we are back to the same problem. But now we have a very simple solution , we can iterate that group of schemas and select the a subset of categories, high enough that the result set is suffiently small , say 20, so that we can start there.

Basically with Orgs, SchemaGroups, Schemas entities we run the party.
And also with those "system" schemas that hold the "hot load" of the category schemas ( the magic truth)


You have X main objectives that I want you to do. Parallize in several agents if / when advisable / possible:
1 - Validate the entire approach, with pros and cons , suggestions and validations (report 1)
2 - Construct the strategy, datamodel and approach (report 2)
3 - Suggest the first 2 levels of categories ( report 3)
4 - Estimation of speed and suggestions of LLM models for each step for some differents cases of documents.












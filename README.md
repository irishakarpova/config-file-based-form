

# Building dynamic form based on configuration scheme.


- React
- Apollo GraphQL
- GraphQL Code Generator
- TypeScript
- Material UI
- FORMIK
- Yup


Great way to organize any task divided into several subtasks.

Here, I would define three task steps:
#### 1. Get graphQL configuration and TypeScript typings out of a GraphQL schema. 

```

schema {
  query: Query
  mutation: Mutation
}
type Query {
  getBannersConfig: FormConfigResult
}
type Mutation {
  createNewBanner(fields: [FormFieldInput]!): [DataField]
}
input FormFieldInput {
  name: String!
  value: String
}
type FormConfigResult {
  fields: [FormField]
}
type FormField {
  name: String!
  label: String
  type: String
  values: [FormFieldValue]
  defaultEmpty: Boolean
  value: String
}
type DataField {
  name: String
  value: String
}
type FormFieldValue {
  value: String
  text: String
}

```

Here, I prefer to use GraphQL Code Generator to get TypeScript typings out of a GraphQL schema. 

```

export interface GetBannersConfig_getBannersConfig_fields {
  __typename: "FormField";
  name: string;
  label: string | null;
  type: string | null;
  values: (GetBannersConfig_getBannersConfig_fields_values | null)[] | null;
  defaultEmpty: boolean | null;
  value: string | null;
}

export interface GetBannersConfig_getBannersConfig {
  __typename: "FormConfigResult";
  fields: (GetBannersConfig_getBannersConfig_fields | null)[] | null;
}

export interface GetBannersConfig {
  getBannersConfig: GetBannersConfig_getBannersConfig | null;
}

export interface CreateNewBanner_createNewBanner {
  __typename: "DataField";
  name: string | null;
  value: string | null;
}

export interface CreateNewBanner {
  createNewBanner: (CreateNewBanner_createNewBanner | null)[] | null;
}

export interface CreateNewBannerVariables {
  fields: (FormFieldInput | null)[];
}

```

Fetch data with the useQuery hook.

```
const { error: configError, data: configData } = useQuery<GetBannersConfig>(GET_CONFIG);

```


#### 2. With Formik library create dynamically rendered form component and set initial values.

```
  const formFieldsConfig:{[index:string]:{
    [type:string]: string
  }} = {};

  const initialValues:{[index:string]:string} = {};

  config.fields.forEach((item: { name: string; type: string;  value: any; }) => {
    initialValues[item.name] = item.value ? item.value : '';
    return formFieldsConfig[item.name] = item;
  })
  
  ```


#### 3. Update data with the useMutation hook.

```
  const [createNewItem, 
    {data, loading: createLoading}] = useMutation<CreateNewBanner>(CREATE_NEW_BANNER);
    
``` 

HandleSubmit calls the mutate function and pass new values to execute the mutation.

```
  const handleOnSubmit = async (values:{[index:string]:string }, setSubmitting:Function) => {
    const fields:Array<{name?:string, value?:string}> = [];
    for (let i in values) {
      if (values.hasOwnProperty(i)) {
        fields.push({
          name: i,
          value: formFieldsConfig[i].type === 'file' ?  file : values[i]
        });
      }
    }
    try{
     await props.createNewItem({
        variables: {fields: fields}
      });
    } catch(e){}
    
    if (setSubmitting){
      setSubmitting(false)
    }
    props.setIsSnack(true)
  };
  
  ```


# Store-API

to install run npm install</br>
then </br>

this project is provided with a dataBase with TOTAL DOCUMENTS: 23

### Ports

> -  The Back end is listening on port 3000.


### Packages

- To install packages: write "npm install" in the terminal.
> -  you can find the dataBase in products.json you can upload to the database by running npm populate.js
- Create .env file and connect it to your DB.


  
### Product Schema

```javascript
const mongoose = require('mongoose')

const productSchema = new mongoose.Schema({
    name:{
        type:String,
        required:[true,'product name must be provided'],
    },
    price:{
        type:Number,
        required:[true,'product price must be provided'],
    },
    featured:{
        type:Boolean,
        default:false,
    },
    rating:{
        type:Number,
        default:4.5,
    },
    createdAt:{
        type:Date,
        default:Date.now(),
    },
    company:{
        type:String,
        enum:{
            values:['ikea','liddy','caressa','marcos'],
            message:`{value} is not supported`,
        }
        //enum:['ikea','liddy,','caressa','marcos']
    }
})

module.exports = mongoose.model('Product',productSchema)
```
### Returning the product searching using $regex

```javascript
    const getAllProducts = async(req,res)=>{
    const {featured,company,name,sort,field,numericFilters} = req.query;
    const queryObject = {}

    if(featured){
        queryObject.featured = featured==='true'?true:false
    }

    if(company){
        queryObject.company = company;
    }
    
    if(name){
        queryObject.name = {$regex:name,$options:'i'}

    }

    if(numericFilters){
        const operatorMap = {
            '>':'$gt',
            '>=':'$gte',
            '=':'$eq',
            '<':'$lt',
            '<=':'$lte',
        }
        const regEx = /\b(<|>|>=|=|<|<=)\b/g

        let filters= numericFilters.replace(
            regEx
            ,(match)=>`-${operatorMap[match]}-`
        )
        const options = ['price','rating'];
        filters = filters.split(',').forEach((item)=>{
            const [field,operator,value] = item.split('-')
            if(options.includes(field)){
                queryObject[field]={[operator]:Number(value)}
            }
        })
    }

    console.log(queryObject)
    let result = Product.find(queryObject)

    //sort
    if(sort){
        const sortList = sort.split(',').join(' ')
        result = result.sort(sortList)
    }else{
        result= result.sort('createAt')
    }

    if(field){
        const fieldsList = fields.split(',').join(' ')
        result = result.select(fieldsList)||10
    }

    const page = Number(req.query.page)||1
    const limit = Number(req.query.limit)
    const skip = (page-1)*limit;
    result = result.skip(skip).limit(limit)

    //our data base got 23 products so 
    //

    const products = await result
    res.status(200).json({products,nbHits:products.length})
}
```

### Postman /products?company=ikea&name=e&sort=-name,price&fields=company, rating
```PRETTY
"products": [
        {
            "featured": false,
            "rating": 4.5,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7ff8",
            "name": "wooden desk",
            "price": 15,
            "company": "ikea",
            "__v": 0
        },
        {
            "featured": false,
            "rating": 4.5,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7ff9",
            "name": "wooden desk",
            "price": 40,
            "company": "ikea",
            "__v": 0
        },
        {
            "featured": false,
            "rating": 4.5,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7ff7",
            "name": "wooden bed",
            "price": 25,
            "company": "ikea",
            "__v": 0
        },
        {
            "featured": false,
            "rating": 4.5,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7ff1",
            "name": "shelf",
            "price": 30,
            "company": "ikea",
            "__v": 0
        },
        {
            "featured": true,
            "rating": 4.5,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7fed",
            "name": "high-back bench",
            "price": 39,
            "company": "ikea",
            "__v": 0
        },
        {
            "featured": false,
            "rating": 4.5,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7feb",
            "name": "emperor bed",
            "price": 23,
            "company": "ikea",
            "__v": 0
        },
        {
            "featured": false,
            "rating": 4.55,
            "createdAt": "2023-06-19T04:01:00.483Z",
            "_id": "648fd2fec03248487c6a7fea",
            "name": "dining table",
            "price": 42,
            "company": "ikea",
            "__v": 0
        }
    ],
    "nbHits": 7
```



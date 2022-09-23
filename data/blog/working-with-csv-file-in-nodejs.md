---
title: 'Working with CSV files in Nodejs'
date: 2023-10-07T15:32:14Z
lastmod: '2023-10-07'
tags: ['nodejs', 'csv', 'csv-parser', 'csv-stringify']
draft: false
summary: 'An overview of how to read and write CSV files in Nodejs.'
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/working-with-csv-file-in-nodejs
---

## Overview

<TOCInline toc={props.toc} exclude="Overview" toHeading={2} />

## Setting up the project

To get started, create a directory `csv_demo` and navigate into it:

```bash
mkdir csv_demo
cd csv_demo
```

Next initialize the project with the following command:

```bash
npm init -y
```

With `-y` flag `npm init` will pick all the default options for the prompts.

Now install the `csv` package:

```bash
npm install csv
```

Following is the sample CSV file we will use for this tutorial:

```csv:csv_demo/read-employees.csv
First Name,Last Name,Email
Bobby,Penatac,bobbypenatac@company.com
Robert,Johnson,robertjohnson@company.com
Scott,Humphrey,scotthumphrey@company.com
```

## Reading CSV file

In this section, you will learn how to read data from a CSV file.

```js:csv_demo/readCSV.js
const { parse } = require('csv-parse');
const fs = require('fs');

fs.createReadStream('./read-employees.csv')
  .pipe(parse({ from_line: 2 }))
  .on('data', function (row) {
    console.log(row)
  });

```

Now execute the script with the following command:

```bash
node readCSV.js
```

Output:

```bash
[ 'Bobby', 'Penatac', 'bobbypenatac@company.com' ]
[ 'Robert', 'Johnson', 'robertjohnson@company.com' ]
[ 'Scott', 'Humphrey', 'scotthumphrey@company.com' ]
```

Here the `parse` method transforms the stream data into an array.

## Writing CSV file

In this section you will learn how to write data into a CSV file.

First lets import the required packages:

```js:csv_demo/writeCSV.js
const fs = require('fs');
const { stringify } = require("csv-stringify");
```

Lets add some dummy data to write into file:

```js:csv_demo/writeCSV.js
...
const employees = [
  {
    firstName: 'Bobby',
    lastName: 'Penatac',
    email: 'bobbypenatac@company.com'
  },
  {
    firstName: 'Robert',
    lastName: 'Johnson',
    email: 'robertjohnson@company.com'
  },
  {
    firstName: 'Scott',
    lastName: 'Humphrey',
    email: 'scotthumphrey@company.com'
  }
];
```

Next create a writeable stream for a while where you want to write the data:

```js:csv_demo/writeCSV.js
...
const filename = "write-employees.csv";
const writableStream = fs.createWriteStream(filename);
```

Then define headers name for the CSV file:

```js:csv_demo/writeCSV.js
...
const columns = [
  'First Name',
  'Last Name',
  'Email'
];
```

At last add the following code to stringify and write data to target CSV file.

```js:csv_demo/writeCSV.js
...
const stringifier = stringify({ header: true, columns: columns });
employees.forEach((employee) => {
  stringifier.write(Object.values(employee));
});
stringifier.pipe(writableStream);
console.log('done');
```

Our full script looks like this:

```js:csv_demo/writeCSV.js
const fs = require('fs');
const { stringify } = require("csv-stringify");

const employees = [
  {
    firstName: 'Bobby',
    lastName: 'Penatac',
    email: 'bobbypenatac@company.com'
  },
  {
    firstName: 'Robert',
    lastName: 'Johnson',
    email: 'robertjohnson@company.com'
  },
  {
    firstName: 'Scott',
    lastName: 'Humphrey',
    email: 'scotthumphrey@company.com'
  }
];

const filename = "write-employees.csv";
const writableStream = fs.createWriteStream(filename);

const columns = [
  'First Name',
  'Last Name',
  'Email'
];

const stringifier = stringify({ header: true, columns: columns });
employees.forEach((employee) => {
  stringifier.write(Object.values(employee));
});
stringifier.pipe(writableStream);
console.log('done!');
```

Run you file in the terminal.

```bash
node writeCSV.js
```

You will see the following output:

```bash
done!
```

Now lets see content in the new file:

```bash
cat write-employees.csv
```

Output:

```bash
First Name,Last Name,Email
Bobby,Penatac,bobbypenatac@company.com
Robert,Johnson,robertjohnson@company.com
Scott,Humphrey,scotthumphrey@company.com
```

# Conclusion

In this article, you learnt how to read and write CSV files using `csv-parser` and `csv-stringify`.
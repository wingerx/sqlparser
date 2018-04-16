Go package for parsing MySQL SQL queries.

## Notice

The backbone of this repo is extracted from [vitessio/vitess](https://github.com/vitessio/vitess).

Inside vitessio/vitess there is a very nicely written sql parser. 
It applies the same LICENSE as vitessio/vitess.

## Usage

```go
import (
    "github.com/wingerx/sqlparser"
)
```

Then use:

```go
sql := "SELECT * FROM t1 WHERE a = 'abc'"
stmt, err := sqlparser.Parse(sql)
if err != nil {
	// Do something with the err
	fmt.Println(err)
	return
}

// Otherwise do something with stmt
switch stmt := stmt.(type) {
case *sqlparser.Select:
	_ = stmt
case *sqlparser.Insert:
}
```

Alternative to read many queries from a io.Reader:

```go
r := strings.NewReader("INSERT INTO t1 VALUES (1, 'a'); INSERT INTO t2 VALUES (3, 4);")

tokens := sqlparser.NewTokenizer(r)
for {
	stmt, err := sqlparser.ParseNext(tokens)
	if err == io.EOF {
		break
	}
	// Do something with stmt or err.
}
```

See [parse_test.go](https://github.com/wingerx/sqlparser/blob/master/parse_test.go) for more examples, or read the [godoc](https://godoc.org/github.com/wingerx/sqlparser).


## Porting Instructions

You only need the below if you plan to try and keep this library up to date with [vitessio/vitess](https://github.com/vitessio/vitess).

### Keeping up to date
```bash
# Update vitess.io & copy sqlparser,bytes2,hack, proto/query, sqltypes to $GOPATH/src/github.com/wingerx/sqlparser 
cd $GOPATH/src/github.com/wingerx/sqlparser
mkdir -p proto

# Copy all the code
cp -pr $GOPATH/src/vitess.io/vitess/go/vt/sqlparser/ .
cp -pr $GOPATH/src/vitess.io/vitess/go/sqltypes .
cp -pr $GOPATH/src/vitess.io/vitess/go/bytes2 .
cp -pr $GOPATH/src/vitess.io/vitess/go/hack .
cp -pr $GOPATH/src/vitess.io/vitess/go/proto/query ./proto/

# Delete some code we haven't ported
cd $GOPATH/src/github.com/wingerx/sqlparser
rm ./sqltypes/arithmetic.go ./sqltypes/arithmetic_test.go ./sqltypes/event_token.go ./sqltypes/event_token_test.go ./sqltypes/proto3.go ./sqltypes/proto3_test.go ./sqltypes/query_response.go ./sqltypes/result.go ./sqltypes/result_test.go

# Fix imports
# vitess.io/vitess/go/vt/proto -> github.com/wingerx/sqlparser/proto
sed -i '.bak' 's_vitess.io/vitess/go/vt/proto_github.com/wingerx/sqlparser/proto_g' *.go
# vitess.io/vitess/go -> github.com/wingerx/sqlparser
sed -i '.bak' 's_vitess.io/vitess/go_github.com/wingerx/sqlparser_g' *.go


# Replace vterrors to fmt/errors
sed -i '.bak' 's/vterrors.Errorf([^,]*, /fmt.Errorf(/g' *.go sqltypes/*.go
sed -i '.bak' 's/vterrors.New([^,]*, /errors.New(/g' *.go sqltypes/*.go
```
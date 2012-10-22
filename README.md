# doozerconfig

## What Is It?

doozerconfig is a Go package for managing json-encoded configuration in doozer. Configuration is directly loaded into a Go struct (see example below). Using struct tags to define the doozer path, the package will automatically read, json-decode and assign the values to the corresponding struct fields. You can also watch for future changes to doozer config and have the struct automatically update.

In future, this package will also provide a way to write configuration back to doozer.

## Installation

```bash
$ go get github.com/srid/doozerconfig
```

## Example

```Go
var Config struct {
    MaxItems  int                `doozer:"config/max_items"`
    DbUri     string             `doozer:"config/db_uri"`
    EnvVars   map[string]string  `doozer:"config/envvars"`
    Verbose   bool   // not in doozer
}

func init() {
    doozer, _ := doozer.Dial("localhost:8046")
    rev, _ := doozer.Rev()
    
    // map Config fields to "/myapp/" + the struct tag above.
    // eg: MaxItems will be mapped to /myapp/config/max_items
    //     EnvVars will be mapped to /myapp/config/envvars/*
    cfg := doozerconfig.New(doozer, rev, &Config, "/myapp/")

    // load config values from doozer and assign to Config fields
    _ = cfg.Load()  

    // watch for live changes to doozer config
    go func() {
        // Monitor automatically updates the fields of `Config`; 
        // the callback function is called for every update or error.
        cfg.Monitor("/myapp/config/*", func(name string, value interface{}, err error) {
            fmt.Printf("config changed in doozer; %s=%v\n", name, value)            
        }) 
    }()
}
```

# Notes

- If a file is deleted from doozer, the config struct is not updated
  (unless it a map entry). Perhaps we should update the value to a
  default value (as specified in a `default` struct tag).

# Changes

- **Oct, 2012**:

  - Support for loading map types

  - API: `Load` now takes doozer revision as a mandatory argument.

  - API: `Monitor` doesn't take a revision argument anymore. Instead
    it uses the one passed to `Load`.

  
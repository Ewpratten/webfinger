# WebFinger

A crate to help you fetch and serve WebFinger resources.

## Examples

Fetching a resource:

```rust
extern crate webfinger;

use webfinger::resolve;

fn main() {
    let res = resolve("acct:test@example.org").expect("Error while fetching resource");

    println!("Places to get more informations about {}:", res.subject);
    for link in res.links.into_iter() {
        println!("- {}", link.href);
    }
}
```

Serving resources:

```rust
extern crate webfinger;

use webfinger::Resolver;

pub struct MyResolver;

impl Resolver<DatabaseConnection> for MyResolver {
    fn instance_domain<'a>() -> &'a str {
        "instance.tld"
    }

    fn find(acct: String, db: DatabaseConnection) -> Result<Webfinger, ResolverError> {
        if let Some(user) = db.find_user_by_name(acct) {
            Ok(Webfinger {
                subject: acct.clone(),
                aliases: vec![acct.clone()],
                links: vec![
                    Link {
                        rel: "http://webfinger.net/rel/profile-page".to_string(),
                        mime_type: None,
                        href: user.profile_url()
                    }
                ]
            })
        } else {
            Err(ResolverError::NotFound)
        }
    }
}

fn main() {
    // Start a web server and map /.well-known/webfinger to a function calling MyResolver::find…
}
```
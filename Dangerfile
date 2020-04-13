require 'find'
require 'image_size'
require 'json-schema'

assets_folder = './blockchains'
allowed_extensions = ['png', 'json']
allowed_file_names = ['info', 'list', 'logo', 'whitelist', 'blacklist']

# Failures

Find.find(assets_folder) do |file|
  file_extension = File.extname(file).delete('.')
  file_name = File.basename(file, File.extname(file))

  # Skip if directory
  if File.directory?(file)
    next
  end

  if !allowed_extensions.include? file_extension
    fail("Extension not allowed for file: " + file)
  end

  if !allowed_file_names.include? file_name
    fail("Filename not allowed for file: " + file)
  end

  # Validate JSON content
  if file_extension == 'json'
    value = nil
    begin
      value = JSON.parse(File.open(file).read)
    rescue JSON::ParserError, TypeError => e
      fail("Wrong JSON content in file: " + file)
    end

    # Validate info.json
    if file_name == 'info'
      schema = {
        "type" => "object",
        "required" => ["name", "website", "short_description", "explorer"],
        "properties" => {
          "name" => {"type" => "string"},
          "website" => {"type" => "string"},
          "short_description" => {"type" => "string"},
          "explorer" => {"type" => "string"}
        }
      }
      errors = JSON::Validator.fully_validate(schema, value)
      errors.each { |error| message("#{error} in file #{file}") } 
    end

    # Validate list.json for validators
    if file_name == 'list'
      schema = {
        "type" => "array",
        "items": {
          "properties": {
              "id": { "type": "string" },
              "name": { "type": "string" },
              "description": { "type": "string" },
              "website": { "type": "string" }
          },
          "required": ["id", "name", "description", "website"]
        }
      }
      errors = JSON::Validator.fully_validate(schema, value)
      errors.each { |error| message("#{error} in file #{file}") } 
    end
  end

  # Validate images
  if file_extension == '.png'
    image_size = ImageSize.path(file)

    if image_size.width > 512 || image_size.height > 512
      fail("Image width or height is higher than 512px for file: " + file)
    end

    if image_size.format == ':png'
      fail("Image should be PNG for file: " + file)
    end

    # Make sure file size only 100kb
    if File.size(file).to_f / 1024 > 100
      fail("Image should less than 100kb for file: " + file)
    end
  end
end

# Image Size

# Warnings

## Mainly to encourage writing up some reasoning about the PR, rather than just leaving a title
#if github.pr_body.length < 1
#  fail "Please provide a summary in the Pull Request description"
#end

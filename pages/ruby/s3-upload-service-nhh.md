# S3 Upload Service

```ruby

require 'aws-sdk'

def with_spinner
  chars = %w[| / - \\]
  not_done = true
  spinner = Thread.new do
    while not_done do
      print chars.rotate!.first
      sleep 0.1
      print "\b"
    end
  end
  yield.tap do         # After yielding to the block, save the return value
    not_done = false   # Tell the thread to exit, cleaning up after itself…
    spinner.join       # …and wait for it to do so.
  end                  # Use the block's return value as the method's
end

# By default aws-sdk looks at these environment variables
# ENV['AWS_REGION'], ENV['AWS_ACCESS_KEY_ID'] and ENV['AWS_SECRET_ACCESS_KEY']

def actual_job
  ENV['AWS_REGION'] = 'us-east-1'
  file_name = 'test.mp4'

  s3 = Aws::S3::Resource.new(region: ENV['AWS_REGION'])
  obj = s3.bucket('s3-bucket-name').object(file_name)
  print "Uploading file #{file_name}"

  obj.upload_file("/Users/bugs/Desktop/#{file_name}")
end

with_spinner do
  actual_job
end

puts "Done!"
```
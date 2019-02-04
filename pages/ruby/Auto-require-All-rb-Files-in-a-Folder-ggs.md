# Auto Require All rb Files In A Folder

```ruby

def require_all_files(path)
  $LOAD_PATH.push path # resource path
  rbfiles = Dir.entries(path).select { |x| /\.rb\z/ =~ x }
  rbfiles -= [File.basename(__FILE__)]
  rbfiles.each do |path|
    require(File.basename(path))
  end
end
```
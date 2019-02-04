# Search Contents Of Files

```ruby

Dir['**/*.html'].each do |path|
  File.open(path) do |f|
    f.grep(/search_string/) do |line|
      puts path, ': ', line
    end
  end
end
```
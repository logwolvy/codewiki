# Excel Reader

```ruby

class ExcelReader
  attr_reader :file_path, :headers, :row_handler, :workbook, :sheet

  def self.call(file_path, headers, &row_handler)
    new(file_path, headers, &row_handler).parse_excel
  end

  def initialize(file_path, headers, &row_handler)
    @file_path = file_path
    @headers = headers
    @row_handler = row_handler
    @workbook = Roo::Spreadsheet.open(file_path)
    @sheet = workbook.default_sheet
  end

  def parse_excel
    process_rows
  end

  private

  def process_rows
    workbook.sheet(sheet).parse(headers.merge(clean: true)).each_with_index do |row, i|
      row_handler.call(
        OpenStruct.new(
          row.merge(
            row_number: i + 1
          )
        )
      )
    end
  end
end
```
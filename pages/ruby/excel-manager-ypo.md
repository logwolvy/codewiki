# Excel Manager

```ruby

class ExcelManager
  attr_accessor :file, :current_user, :custom_class, :record

  def initialize(file, current_user, entity)
    @file = file
    @current_user = current_user
    @updated_records = 0
    @new_records = 0
    @error_records = 0
    @error_rows = []
    @records = []
    @custom_class = class_mapper[entity]
  end

  def import_excel
    spreadsheet = Roo::Spreadsheet.open(file.path)
    if spreadsheet.last_row <= 1
      raise Exceptions::ServiceFailure, "File is empty"
    end
    header = excel_header(spreadsheet)
    process_excel_sheet(spreadsheet, header)
    response
  end

  private

  def class_mapper
    {
      products: ProductsExcelManager,
      cus***REMOVED***s: Cus***REMOVED***sExcelManager,
      drivers: DriversExcelManager
    }
  end

  def excel_header(spreadsheet)
    spreadsheet.row(1).map { |h| h.remove(" ").underscore }
  end

  def process_excel_sheet(spreadsheet, header)
    ActiveRecord::Base.transaction do
      (2..spreadsheet.last_row.to_i).map do |row_no|
        @row = Hash[[header, spreadsheet.row(row_no)].transpose]
        @row_no = row_no
        @other_errors = nil
        @current_class = custom_class.new(current_user, @row)
        @records = @current_class.excel_row
        process_records_and_errors
      end
    end
  end

  def process_records_and_errors
    if @current_class.respond_to?("other_errors")
      @other_errors = @current_class.other_errors
    end
    save_record
    check_errors if @errors.present? || @other_errors.present?
  end

  def save_record
    @errors = []
    @records.map do |record|
      if record.valid? && @other_errors.blank?
        associated_record = @records.size > 1 && record == @records[-1]
        update_record_counts(@records[-1]) unless associated_record
        record.save! if records_valid?
      else
        @errors << record.errors.full_messages
      end
    end
  end

  def check_errors
    @error_records += 1
    @errors << @other_errors if @other_errors.present?
    @error_rows << { row_no: @row_no, error: @errors.flatten }
  end

  def update_record_counts(record)
    return unless records_valid?
    if record.new_record?
      @new_records += 1
    else
      @updated_records += 1
    end
  end

  def response
    {
      new_records_count: @new_records,
      updated_records_count:  @updated_records,
      error_records_count: @error_records,
      error_rows: @error_rows
    }
  end

  def records_valid?
    flags = @records.map(&:valid?)
    flags.all? { |flag| flag == true }
  end
end
```
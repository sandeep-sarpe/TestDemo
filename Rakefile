namespace :db do
 require "sequel"
 require "pg"
 Sequel.extension :migration


 # Database configuration
 data=Hash[File.read('database/conf/db_config.config').split("\n").map{|i|i.split('=')}]
 sa_database=data["sa_pg_database"]

 pg_host=ENV['PG_HOST']
 pg_port=ENV['PG_PORT']
 pg_username=ENV['PG_USERNAME']
 pg_password=ENV['PG_PASSWORD']

 db_URL="postgres://#{pg_username}:#{pg_password}@#{pg_host}:#{pg_port}/#{sa_database}"

 db = Sequel.connect(db_URL) 

 # Default task
 desc "This is default task"
 task :default do
 end

 # migrate task - apply Schema changes
 desc "Run migrations: Applying schema changes"
 task :migrate, [:version] do |t, args|
  puts "Applying schema changes...."
  if args[:version]
   puts "Migrating to version #{args[:version]}"
   Sequel::Migrator.run(db, "database/schemas/migrations", target: args[:version].to_i)
  else
   Sequel::Migrator.run(db, "database/schemas/migrations")
   puts "Migrated to latest."
  end
 end

 # recreate_functions task - recreate functions
 desc "Recreating functions"
 task :recreate_functions do
  puts "Recreating functions...."
  puts " Running ETL functions..."
  Dir.foreach('database/schemas/functions/ETL') do |item|
   next if item == '.' or item == '..'
    db.run File.read("database/schemas/functions/ETL/#{item}")
    puts "  Recreated #{item}" 
   end
   puts "Recreated functions." 
 end

 # load_tests task - recreates test functions (testing purpose only)
 desc "Recreating test functions"
 task :load_tests do
  puts "Recreating test functions...."
  Dir.foreach('test/testsuites') do |item|
   next if item == '.' or item == '..'
    db.run File.read("test/testsuites/#{item}")
    puts "  Recreated testsuite function - #{item}" 
   end
  Dir.foreach('test/tests') do |item|
   next if item == '.' or item == '..'
    db.run File.read("test/tests/#{item}")
    puts "  Recreated test function - #{item}" 
   end
  Dir.foreach('test/globalhelp') do |item|
   next if item == '.' or item == '..'
    db.run File.read("test/globalhelp/#{item}")
    puts "  Recreated global help function - #{item}" 
   end
   puts "Recreated functions." 
 end
 
 # run_tests task - runs tests (testing purpose only)  
 desc "Running tests"
 task :run_tests do
  puts "Running tests...."
  db.run "SELECT test.testsuite_etl()"
 end

 # evaluate_test_result task - evaluate test result (testing purpose only)  
 desc "Evaluating test result...."
 task :evaluate_test_result do
  puts "Evaluating test result...."
  db.fetch("select test_batch from test.test_results order by test_batch desc limit 1"){|r| 
  p r
  value = "'"+r[:test_batch]+"'"	
  db.run "SELECT test.evaluate_test_result(#{value})"
  }
  puts "Hurrey! All tests are PASSED!"
 end 

end

require 'open3'
require 'rake/clean'

PROBLEM_NAME = "problem name"
ROUND_ID = 0
TESTER = "tester.jar"
SEED = 1

CLEAN.include %w(data/* *.gcda *.gcov *.gcno *.png)

desc 'c++ file compile'
task :compile do
  sh("g++ -std=c++11 -D__NO_INLINE__ -W -Wall -Wno-sign-compare -O2 -o #{PROBLEM_NAME} #{PROBLEM_NAME}.cpp")
end

desc 'exec and view result'
task run: [:compile] do
  sh("java -jar ./#{TESTER} -vis -seed #{SEED} -exec './#{PROBLEM_NAME}'")
end

desc 'check single'
task one: [:compile] do
  if ENV["debug"]
    sh("time java -jar #{TESTER} -seed #{SEED} -debug -novis -exec './#{PROBLEM_NAME}'")
  else
    sh("time java -jar #{TESTER} -seed #{SEED} -novis -exec './#{PROBLEM_NAME}'")
  end
end

desc 'check for windows'
task windows: [:compile] do
  sh("java -jar ./#{TESTER} -novis -seed #{SEED} -exec ./#{PROBLEM_NAME}.exe")
end

desc 'check out of memory'
task :debug do
  sh("g++ -std=c++11 -W -Wall -g -fsanitize=address -fno-omit-frame-pointer -Wno-sign-compare -O2 -o #{PROBLEM_NAME} #{PROBLEM_NAME}.cpp")
  sh("time java -jar #{TESTER} -seed #{SEED} -novis -exec './#{PROBLEM_NAME}'")
end

desc 'check how many called each function'
task :coverage do
  sh("g++ -W -Wall -Wno-sign-compare -o #{PROBLEM_NAME} --coverage #{PROBLEM_NAME}.cpp")
  sh("time java -jar #{TESTER} -seed #{SEED} -novis -exec './#{PROBLEM_NAME}'")
end

desc "example test"
task sample: [:compile] do
  run_test(from: 1, to: 10)
end

desc "production test"
task test: [:compile] do
  run_test(from: 1000, to: 1100)
end

desc "system test"
task final: [:compile] do
  run_test(from: 2001, to: 3000)
end

def run_test(from:, to:)
  File.open("result.txt", "w") do |file|
    from.upto(to) do |seed|
      puts "seed = #{seed}"
      file.puts("----- !BEGIN! ------")
      file.puts("Seed = #{seed}")

      data = Open3.capture3("time java -jar #{TESTER} -seed #{seed} -novis -exec './#{PROBLEM_NAME}'")
      file.puts(data.select { |d| d.is_a?(String) }.flat_map { |d| d.split("\n") })
      file.puts("----- !END! ------")
    end
  end

  ruby "scripts/analyze.rb #{to - from + 1}"
end

task default: :compile

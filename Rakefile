require "ostruct"
require "yaml"
require "erb"

REPO_URL = "https://charts.darkedges.com"
WEB_ROOT = "public"

class Chart < OpenStruct
  def initialize(source)
    @to_str = source
    super(YAML.load_file(source))
  end

  def self.all
    @all ||= Dir["../../forgerock/devspace-forgerock-quickstart/helm/**/Chart.yaml"].map { |source| new(source) }.select { |line| line.version == "1.0.0"}
  end

  def self.targets
    all.map(&:target)
  end

  def self.from_target(target)
    all.detect { |chart| target == chart.target }
  end

  attr_reader :to_str

  def target
    "docs/#{name}-#{version}.tgz"
  end
end

desc "Build packaged helm charts"
task package: Chart.targets

rule ".tgz" => ->(target) { Chart.from_target(target) } do |t|
  sh "cd ../../forgerock/devspace-forgerock-quickstart/helm/#{t.source.name} && helm dependency build"
  sh "helm package -d . ../../forgerock/devspace-forgerock-quickstart/helm/#{t.source.name}"
end

task index: "./index.yaml"
rule "./index.yaml" => Chart.targets do
  sh "helm repo index . --url https://charts.darkedges.com"
  File.write("./index.html", ERB.new(File.read("index.html.erb")).result)
end

task :clean do
  sh "rm -rf *.tgz && rm -f index.html && rm -f index.yaml"
end

task default: [:package, :index]
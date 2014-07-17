#!/usr/bin/env rake
require "bundler/gem_tasks"

require 'rake'
require 'rspec/core/rake_task'

desc "Run all examples"
RSpec::Core::RakeTask.new(:spec) do |t|
  t.rspec_opts = %w[--color]
end

task :default => [:spec]

require 'csv'
require 'pry'
require_relative 'lib/countries'

namespace :countries do
  class ::Country
    [:min_longitude, :min_latitude, :max_longitude, :max_latitude].each do |new_attr|
      define_method("#{new_attr}=") do |value|
        @data[new_attr.to_s] = value
      end
    end

    def to_hash
      hash = {}
      AttrReaders.each do |a|
        if a == :currency
          hash[a.to_s] = self.currency.nil? ? '' : self.currency.code
        else
          hash[a.to_s] = self.send(a.to_s)
        end
      end

      hash
    end
  end

  task :import_country_bound_boxes do
    raise ArgumentError.new('Missing COUNTRY_BOUND_BOXES_CSV') if ENV['COUNTRY_BOUND_BOXES_CSV'].empty?

    updated_countries = {}
    File.open(File.join(File.dirname(__FILE__), 'lib', 'data', 'countries_with_boundary_boxes.yaml'), 'w') do |countries_file|
      countries_bound_boxes = CSV.read(ENV['COUNTRY_BOUND_BOXES_CSV'], headers: true, header_converters: :symbol)
      countries_bound_boxes.each do |country_bound_boxes|
        country_alpha2 = country_bound_boxes[:alpha_2]
        country = Country.new(country_alpha2)

        if country.nil?
          puts "Country #{country_alpha2} not found on countries gem dataset."
          next
        end

        country.min_longitude = country_bound_boxes[:min_lon]
        country.min_latitude = country_bound_boxes[:min_lat]
        country.max_longitude = country_bound_boxes[:max_lon]
        country.max_latitude = country_bound_boxes[:max_lat]

        updated_countries[country_alpha2] = country.to_hash
      end

      # Add missing countries from countries.yaml
      CountriesData = YAML.load_file(File.join(File.dirname(__FILE__), 'lib', 'data', 'countries.yaml'))
      CountriesData.each do |country_data|
        unless updated_countries.has_key?(country_data[0])
          puts "Country #{country_data[0]} not found on country boundary boxes dataset."
          missing_country = Country.new(country_data[0])
          updated_countries[country_data[0]] = missing_country.to_hash
        end
      end

      countries_file.write(updated_countries.to_yaml)
    end
  end
end

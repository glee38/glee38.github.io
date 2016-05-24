---
layout: single
title: My First CLI Gem
categories: Development
comments: true
draft: false
---
For this project, I was challenged to build a CLI gem from scratch. Although the project seemed intimidating at first, the CLI Gem video walkthrough was extremely helpful in giving me the push that I needed to begin building my gem. Below, I have included some notes that were taken from the video, as well as my personal thought process, struggles, and triumphs. You will witness the rewarding fruits of coding labor, and share in the birth of the Teavana-Cli-Gem (which I've started to fondly refer to as my first code child). And while my gem could definitely still use some refactoring and better code structure, I am so excited to share with you what I have learned thus far.

<!--more-->

## How to build a CLI gem

### Here is a quick breakdown of how to get started:

1. Plan your gem, imagine your interface
2. Start with the project structure - google: "how to build a ruby gem"
3. Start with the entry point - the file run
4. Force that to build the CLI interface
5. Stub out the interface
6. Start making things real
7. Discover objects
8. Program

### Step 1: Plan your gem, imagine your interface

What kind of gem are you looking to make? How do you want the user to interact with the gem via the CLI?

I think the best way I found to answer these questions, was to ask myself what kind of program I wish already existed. 

I am a huge tea fanatic, and I wanted to create an easy way to look up different types of tea, their ingredients, availability, etc. Once I had my idea in place, I browsed various tea websites to determine which site would best suit the needs of my gem. I decided to go with Teavana (my go-to tea store), since I generally purchased most of my tea from them. 

Now, it was time to plan the UI. 

For my Teavana-CLI-Gem, I began by writing out the general interface design as follows:

```
Home
Welcome to Teavana!
What kind of tea are you looking for today?
1. Green Tea
2. Black Tea
3. White Tea
...etc.

#user enters "1"
Home > Green Tea
Here are a list of green teas available:
1. Organic Imperial Matcha Singles: 10-Pack - 19.95
2. Organic Peach Matcha Singles: 10-Pack - 19.95
3. Gyokuro Imperial Green Tea - 19.98
...etc.
Which tea would you like to know more about?
Enter "home" to go back to the full list of teas.

#user enters "3"
Home > Green Tea > Gyokuro Imperial Green Tea

RATING
4.4/5 stars

PRICE
19.98

DESCRIPTION

Gyokuro bushes are covered in shade two weeks before 
harvesting, which creates a light but very complex 
blend and a luscious deep dark green color. The 
shading helps the leaves to retain chlorophyll, 
which concentrates both the green tea taste and 
nutrients, making this a bright and flavorful favorite.

TASTING NOTES
Rich, almost full-bodied, smooth taste with sweet 
ending and complex notes

CAFFEINE LEVEL
3

CAFFEINE GUIDE
4: 40+ MG
3: 26-39 MG
2: 16-25 MG
1: 1-15 MG
0: CAFFEINE-FREE

INGREDIENTS
Green tea
```

### Step 2: Start with the project structure

Google "How to build a Ruby gem" for a list of instructions/guide.

**TIP: You can create a new gem with Bundler. In your command line, type:**

```
bundle gem teavana-cli-gem
```

This stubs out all of the structure for you in advance!

**P.S.** When you first run Bundler, it will ask you if you want to license your gem. I said "hells yes".

After a few minor changes, additions in dependencies and gem files, the environment set-up for my gem was complete. Here is the final breakdown of my file structure:

```
teavana-cli-gem
  bin
    console
    setup
    teavana # program is run from this file
  lib
    teavana_cli_gem
      cli.rb
      teas.rb
      teascraper.rb
      version.rb
    teavana_cli_gem.rb
  spec
    .gitignore
    .rspec
    .travis.yml
    Gemfile
    Gemfile.lock
    LICENSE.txt
    NOTES.md # notes about my interface design
    Rakefile
    README.md
    teavana_cli_gem.gemspec
```

### Steps 3-5: Start with the entry point - the file run, Force that to build the CLI interface, Stub out the interface

Once I had my project structure and had mapped out my interface, these steps were almost a given. I was able to plow these steps extremely quickly, and jumped into:

### Step 6-8: Start making things real, discover objects, program

I began by looking over my `NOTES.md`, and went from there. The first step of my gem was to do the following:

```
Home
Welcome to Teavana!
What kind of tea are you looking for today?
1. Green Tea
2. Black Tea
3. White Tea
...etc.
```

It became clear that I needed to build a method that would scrape the list of teas from Teavana's index page, and output the result to the CLI. At this point, I knew I would need to build at least two classes, `TeavanaCliGem::CLI` and `TeavanaCliGem::TeaScraper`.

For this function, I came up with the following method in the `TeavanaCliGem::TeaScraper` class:

```ruby
  def self.scrape_tea_types
    # scrape from teavana website index page
    index_url = "http://www.teavana.com/us/en/tea"
    doc = Nokogiri::HTML(open(index_url))

    doc.css("ul#by-type li").each do |type| 
    # selects teas 'BY TYPE'
      # tea.text => "Green"
      @@tea_types << type.text unless @@tea_types.include?(type.text)
    end
    @@tea_types
  end
```

Now, the above method worked fine, but once I got my entire gem to work smoothly, I went back and refactored this method to the following:

```ruby
  def self.scrape_tea_types
    index_url = "http://www.teavana.com/us/en/tea"
    doc = Nokogiri::HTML(open(index_url))
    @tea_types = doc.css("ul#by-type li").collect{|type| type.text} 
  end
```

There was no reason for me to use the #each method, when I was explicitly forcing my method to return an array. Why not just use the #collect method, whose implicit return value is just that?

Once I had sucessfully scraped the different tea types, I wrote a method with which to list them using index numbers:

```ruby
  def self.list_tea_types
    scrape_tea_types
    @tea_types.each.with_index(1) do |tea,i|
      puts "#{i}. #{tea}"
    end
  end
```

Finally, I implemented these methods into the `TeavanaCliGem::CLI` class, which would ultimately use the method `#call` to run the program from `./bin/teavana`:

```ruby
  def call
    puts "Welcome to " + "Teavana".colorize(:green) + "!"
    # more code to come
  end
  
  def tea_types # lists types of tea
    puts "Here is our menu of available types of tea:".colorize(:yellow)
    puts "Home > Tea".colorize(:red)
    puts " "
    @teas = TeavanaCliGem::TeaScraper.list_tea_types
    puts " "
    puts "Please enter the number of the tea you are interested in.".colorize(:cyan)
    puts "For example, if you would like to view our menu of #{@teas[0]} Teas, enter '1'.".colorize(:cyan)
  end
```

Next, I moved on to the following part of my interface:

```
#user enters "1"
Home > Green Tea
Here are a list of green teas available:
1. Organic Imperial Matcha Singles: 10-Pack - 19.95
2. Organic Peach Matcha Singles: 10-Pack - 19.95
3. Gyokuro Imperial Green Tea - 19.98
...etc.
Which tea would you like to know more about?
Enter "home" to go back to the full list of teas.
```

The CLI would need to:

1. ask for the user's input 
2. select a type of tea based on that input and
3. display a list of the specific tea kinds that were available for that tea type.

In order to achieve this, I built the following method in the `TeavanaCliGem::TeaScraper` class:

```ruby
def self.scrape_tea_urls
    # some method to get the href of selected tea
    # @tea_urls = shovel tea type urls into this array
    
    index_url = "http://www.teavana.com/us/en/tea"
    doc = Nokogiri::HTML(open(index_url))

    doc.css("ul#by-type li a").each do |type| # selects the 'a' element
      @tea_urls << type["href"] unless @tea_urls.include?(type["href"])
    end
    @tea_urls
  end
```

This method was also refactored at the end to the following:

```ruby
  def self.scrape_tea_urls
    index_url = "http://www.teavana.com/us/en/tea"
    doc = Nokogiri::HTML(open(index_url))
    
    @tea_urls = doc.css("ul#by-type li a").collect{|type| type["href"]}
  end
```

This method would grab the url's of each tea type, and shovel them into an array. For example: 

```
["http://www.teavana.com/us/en/tea/greentea", 
 "http://www.teavana.com/us/en/tea/blacktea", 
  etc.]
```

From here, I built a scraper method, `#scrape_specific_tea_kinds(input)` that would call on the `@tea_urls` array, select one of these urls (based on the user's input), and plug it in to set the value of the `index_url` as follows:

```ruby
def self.scrape_specific_tea_kinds(input)
  # calls on @tea_urls array and selects appropriate index_url using index #
    # index_url = user's number input   
    scrape_tea_urls

    index_url = @tea_urls[input.to_i-1] + 
    "?sz=1000&start=0&lazyload=true&format=ajax" 
    # preloads entire page
    doc = Nokogiri::HTML(open(index_url))

    @specific_tea_kinds = doc.css(".product_card").collect {|card| card.css(".name").text} 
  end
```

This method would scrape the names of all the different kinds of tea for the selected tea type. For example, if you were to select `Green` tea, then this method would use `"http://www.teavana.com/us/en/tea/greentea"` as the index url and scrape the kinds of green tea available:

```
1. Green Tea Favorites
2. Organic Imperial Matcha Singles: 10-Pack
3. Organic Peach Matcha Singles: 10-Pack
4. Organic Chai Matcha Singles: 10-Pack
5. Gyokuro Imperial Green Tea
6. Emperor's Clouds and Mist® Green Tea
...etc.
```

(**NOTE**: This was the final result of the method above, however, towards the end of my gem, I realized that Teavana had lazy loaders, which meant that the teas that were loaded via the lazy loader were not being accounted for through open-uri, since it did not read Javascript. At first, I felt discouraged, because by the time I had realized this, my entire gem was already working quite smoothly. In retrospect, however, I'm really glad that this happened because I was able to learn about new gems and new tricks to either 1. read the javascript or 2. preload the entire webpage. Ultimately, I chose to preload the page with Avi's help by adding `"?sz=1000&start=0&lazyload=true&format=ajax"` to my base url. This worked like a charm.)

Then, I built the `#list_specific_tea_kinds` method to list the teas with their index numbers:

```ruby
  def self.list_specific_tea_kinds
    @specific_tea_kinds.each.with_index(1) do |tea,i|
      puts "#{i}. #{tea}"
    end
  end
```

Finally, I added the following code in `TeavanaCliGem::CLI` class:

```ruby

  def list_and_select_tea_kind
    TeavanaCliGem::TeaScraper.scrape_tea_urls
    TeavanaCliGem::TeaScraper.scrape_specific_tea_kinds(@input_1)
    @tea_kinds = TeavanaCliGem::TeaScraper.list_specific_tea_kinds
    select_tea_kind
  end
```

```ruby
def select_tea_type # selects type of tea - Green, Black, etc.
    tea_types

    begin
      @input_1 = gets.strip
      if @input_1.to_i > 0 && @input_1.to_i <= @teas.size
        puts " "
        puts "You have selected #{@teas[@input_1.to_i-1]} Tea.".colorize(:yellow)
        puts "Home > Tea > ".colorize(:red) + "#{@teas[@input_1.to_i-1]} Tea".colorize(:red).underline
        puts " "
        list_and_select_tea_kind
      elsif @input_1.downcase == "exit"
        puts "Please type 'exit' again to confirm.".colorize(:magenta) 
        # need to type 'exit' again to actually exit out of loop
        # only need to type it twice if user types 'home' from #select_tea_kind loop and then tries to exit
        # cannot figure out why when @input_1 is clearly 'exit' - used the 'break' keyword here to break out of the loop and it still didn't work
        # tried multiple ways to make it break out, none worked
      else
        puts "Oops! We are not sure what you were looking for. Please type 'home' to go back to the menu of available teas or 'exit'.".colorize(:magenta)
      end
      break if @do_break
    end while @input_1 != "exit"
  end
```

```ruby
  def select_tea_kind # selects specific kinds of tea -
   Dragonwell Green Tea, etc
    puts " "
    puts "Which tea would you like to know more about?".colorize(:cyan)
    puts "For example, if you would like more details on #{@tea_kinds[0]}, enter '1'.".colorize(:cyan)
    puts "(Enter 'home' to go back to the menu of all available teas or 'exit'.)".colorize(:magenta)

    begin
      @input_2 = gets.strip

      if @input_2.to_i > 0 && @input_2.to_i <= @tea_kinds.size
        puts " "
        puts "You have selected #{@tea_kinds[@input_2.to_i-1]}.".colorize(:yellow)
        puts "Home > Tea > #{@teas[@input_1.to_i-1]} Tea > ".colorize(:red) + "#{@tea_kinds[@input_2.to_i-1]}".colorize(:red).underline
        puts " "
      list_tea_details
      puts "(Enter 'back' to go back to the menu of #{@teas[@input_1.to_i-1]} Teas, 'home' to go back to the menu of all available teas, or 'exit'.)".colorize(:magenta)
      elsif @input_2.downcase == "back"
        puts " "
        puts "Home > Tea > ".colorize(:red) + "#{@teas[@input_1.to_i-1]} Tea".colorize(:red).underline
        puts " "
        list_and_select_tea_kind
      elsif @input_2.downcase == "home"
        puts " "
        select_tea_type
      elsif @input_2.downcase == "exit"
        @do_break = true # flag to break out of parent loop #select_tea_type 
      else
        puts "Oops! We are not sure what you were looking for. Please type 'home' to go back to the menu of available teas or 'exit'.".colorize(:magenta)
      end
    end while @input_2 != "exit"
  end
```

This part was tricky, because I had to find a way to break out of the nested loops if the user chose to exit the program while inside the inner loop. To achieve this, I added a flag inside the inner loop that would turn `true` if the user typed "exit". The outer loop would respond by breaking the loop by `break if @do_break`. 

I also had to account for the fact that the user might accidently type invalid input, so I made sure to add that conditional to the `if` statements of both `#select_tea_type` and `#select_tea_kind`. The input would only be valid if it was a number greater than one, but less than or equal to the size of the array in which the teas were stored. 

The final bug that I found at this point in my code, was that if I went inside of my inner loop (of specific tea kinds), typed `"home"`, and tried to exit from "home" (which lists the tea types like Green, Black, etc.), then my program required me to type `"exit"` again before it actually exited out of the outer loop. I could not figure out how to fix this bug, since `input_1` was clearly `==` to `"exit"`. Ultimately, I ended up just asking the user to confirm by typing "exit" again, upon which the loop did successfully break.

The next step was to implement the following part of my interface:

```
#user enters "3"
Home > Teas > Green > Gyokuro Imperial Green Tea

RATING
4.4/5 stars

PRICE
19.98

DESCRIPTION

Gyokuro bushes are covered in shade two weeks before 
harvesting, which creates a light but very complex 
blend and a luscious deep dark green color. The 
shading helps the leaves to retain chlorophyll, 
which concentrates both the green tea taste and 
nutrients, making this a bright and flavorful favorite.

TASTING NOTES
Rich, almost full-bodied, smooth taste with sweet 
ending and complex notes

CAFFEINE LEVEL
3

CAFFEINE GUIDE
4: 40+ MG
3: 26-39 MG
2: 16-25 MG
1: 1-15 MG
0: CAFFEINE-FREE

INGREDIENTS
Green tea
```

Before writing the code to accomplish this, I had to account for the following:

1. If the tea went on sale - how would that affect the code where the price was being scraped from?
2. If any of the above attributes were unavailable
3. How to scrape from just the one url that was associated with the specific tea chosen
4. Take the user's input both the first AND second time, and use them together to access the appropiate specific tea

Once I had these conditions in mind, I went ahead and began to write the code to achieve the above. In the `TeavanaCliGem::TeaScraper` class, I built the following method:

```ruby
def self.scrape_specific_tea_kinds_urls(input1)
    scrape_tea_urls
    index_url = @tea_urls[input1-1] + "?sz=1000&start=0&lazyload=true&format=ajax"
    doc = Nokogiri::HTML(open(index_url))
    # @specific_tea_kinds_urls = array of urls for specific tea kinds for a single type of tea. 
    Ex. Green => [url for matcha, url for dragon pearl]

    @specific_tea_kinds_urls = doc.css(".product_card .name a").collect{|card| card["href"]} 
    # url of each specific tea card (leads to tea details)
  end
```

This method would scrape the tea urls, take in an argument of the user's first input to select the appropriate url, set it as the `index_url`, and scrape the urls of each specific kind of tea for the chosen tea type.

Next, I built the following method:

```ruby
def self.scrape_tea_details(input2)
    @tea_details = {}
    index_url = @specific_tea_kinds_urls[input2-1] 

    doc = Nokogiri::HTML(open(index_url))
      price = "N/A"
      availability = "N/A"
      description = "N/A"
      tasting_notes = "N/A"
      caffeine_level = "N/A"
      ingredients = "N/A"

      price = doc.css(".pdp-price-div").css("div[itemprop]").text.gsub(/\t/,'').gsub(/\n/,'').gsub(/\r/,'')
      
      availability = doc.css(".pdp-avail").text.gsub(/\n/,'')

      description = doc.css("div#longdesc.open").text.gsub(/\n/,'').gsub(/\r/,'')
      
      unless doc.css("span.pdp-value.open").size == 0
        tasting_notes = doc.css("span.pdp-value.open").text
      end

      unless doc.css("input.caffeineLeveltxt").size == 0
        caffeine_level = doc.css("input.caffeineLeveltxt").attribute("value").value
      end

      unless doc.css(".ingredients.pdp-product-info").size == 0
        ingredients = doc.css(".ingredients.pdp-product-info").children[-2].text
      end
      
      @tea_details = {:price => price, :availability => availability, :description => description, :tasting_notes => tasting_notes, :caffeine_level => caffeine_level, :ingredients => ingredients} 
      
      @tea_details 
  end
```

This method take in an argument of the user's second input to select the appropriate url in `@specific_tea_kinds_urls`, set it as the `index_url`, scrape the necessary information, and set them as key/value pairs inside the `@tea_details` hash.

This part of the process went pretty smoothly, with the exception of the "rating" attribute. Unfortunately, I was not able to scrape that piece of data, as the website had set the rating inside a span class that I was not able to access. After a few hours, I decided it was best to move on and leave that attribute out for the time being.

Once I had finished building my `TeavanaCliGem::TeaScraper` class, I figured that the best way to mass assign the scraped attributes would be through a separate `TeavanaCliGem::Teas` class:

```ruby
class TeavanaCliGem::Teas
  attr_accessor :name, :availability, :price, :description, :tasting_notes, :caffeine_level, :ingredients

  @@all = []

  def initialize(tea_attributes_hash) # take in an argument of the TeaScraper class
    self.add_tea_attributes(tea_attributes_hash)
    @@all << self unless @@all.include?(self)
  end

  def add_tea_attributes(tea_attributes_hash)
    tea_attributes_hash.each do |k,v|
      send("#{k}=", v) unless v == nil
    end
    self
  end

  def self.all
    @@all
  end
end
```

This class was responsible for creating the different types of teas, adding their attributes, and saving them to an array just in case I wanted to access them later.

Finally, it was time to implement these methods and collaborate the classes within my cli by adding the following methods into the `TeavanaCliGem::CLI` class:

```ruby
  def list_tea_details
    TeavanaCliGem::TeaScraper.scrape_specific_tea_kinds_urls(@input_1.to_i)
    tea_details_hash = TeavanaCliGem::TeaScraper.scrape_tea_details(@input_2.to_i)
    tea = TeavanaCliGem::Teas.new(tea_details_hash)

    puts "PRICE".colorize(:blue)
    puts "#{tea.price}"
    puts "#{tea.availability}"
    puts " "
    puts "DESCRIPTION".colorize(:blue)
    puts "#{tea.description}"
    puts " "
    puts "TASTING NOTES".colorize(:blue)
    puts "#{tea.tasting_notes}"
    puts " "
    puts "CAFFEINE LEVEL".colorize(:blue)
    puts "#{tea.caffeine_level}"
    puts " "
    caffeine_guide
    puts " "
    puts "INGREDIENTS".colorize(:blue)
    puts "#{tea.ingredients}"
    puts " "
  end
  
  def caffeine_guide
    puts "CAFFEINE GUIDE".colorize(:blue)
    puts <<-DOC.gsub /^\s+/, ""
    4: 40+ MG
    3: 26-39 MG
    2: 16-25 MG
    1: 1-15 MG
    0: CAFFEINE-FREE
    DOC
  end
```

The above method was responsible for printing out a tea's attributes via

1. `TeavanaCliGem::TeaScraper.scrape_specific_tea_kinds_urls(@input_1.to_i)` 
2. then setting
    `tea_details_hash` equal to `TeavanaCliGem::TeaScraper.scrape_tea_details(@input_2.to_i)`
3. and finally, instantiating a tea with the hash as an argument:
    `tea = TeavanaCliGem::Teas.new(tea_details_hash)`

(**NOTE**: Although my CLI looks pretty extensive and a bit too long, I felt that it was necessary in order to give the user as many options as possible. If I can figure out a better way to set up this part of my code in the future, I would love to go back and refactor some of the CLI to make it look cleaner.)

FINALLY, after many hours of labor, my baby, `TeavanaCliGem` was born on **Tuesday, March 15th, 2016 at 2:05 pm**. At a whopping 0 oz., my very first code child reared its beautifully coded head into my terminal, bringing great joy and overwhelming pride. Hallelujah!!!!!

If you would like to view the final product in action, [here](https://www.youtube.com/watch?v=gX31GLx8Org) is a video walkthrough of the gem.

If you would like to share in my happiness, please feel free to download my baby by typing in the following:

```
(Going to publish the gem after the pairing session in
 order to ensure the best possible results/code)
```

<h1>A step by ste guide on how to start a Test driven Development with rspec</h1>
<p>Based on Thougthbot & El Taco Lab methodology</p>

<h2>Install rspec gem</h2>
<p>In your gemfile add the following code:</p>

```ruby
group :development, :test do
  gem 'rspec-rails', '~> 3.0'
end
```
<p>Bundle install:</p>

```ruby
bundle install
```
<p>Generate Rspec files</p>

```ruby
rails generate rspec:install
```

<h2>Create your first feature Rspec test</h2>
<p>In this example from Thougthbot, we will test the feature 'Submiting a link post' as in Reddit.</p>
<p>First we write pseudo code:</p>

```ruby
As a user
When I visit the home page
And I click "Submit a link post"
And I fill in my title and URL
And I click "Submit"
Then I should see the title on the page
And it should link to the given URL
```
<h3>Capybara</h3>
<p>In order to test the features we need it to be able to interact with the browser, for that we need to install Capybara</p>

<p>Add the gem to your gemfile:</p>

```ruby
gem "capybara"
```
<p>And require it from tour <code>rails_helper</code>file</p>

```ruby
require "capybara/rails"
```

<p>! If you encounter an issue like "Capybara--rails not defined", try requiring as follow: </p>

```ruby
require "capybara/rspec"
```

<h3>The test</h3>
<p>We create the test in <code># spec/features/user_submits_a_lin_spec.rb</code></p>
<p>It is important to name your files with "spec.rb" as rspec will run all the files finishing with it.</p>
<p>Following is the typical code to test a scenario within a feature, each feature can have several scenarios</p>

```ruby
require "rails_helper"

# As a user
RSpec.feature "User submits a link" do
  scenario "they see the page for the submitted link" do
    link_title = "This Testing Rails book is awesome!"
    link_url = "http://testingrailsbook.com"
#  When I visit the home page
    visit root_path
#  And I click "Submit a link post"
    click_on "Submit a new link"
#  And I fill in my title and URL
    fill_in "link_title", with: link_title
    fill_in "link_url", with: link_url
#  And I click "Submit"
    click_on "Submit!"
#  Then I should see the title on the page
# And it should link to the given URL
    expect(page).to have_link link_title, href: link_url
  end
end
```

<h3>Running the test</h3>
<p>To run the test you must run it on your console with: <code>rspec</code></p>
<p>Now you can change your code depending on the error messages displayed in your terminal.</p>

<h2>Invalid Scenarios</h3>
<p>Some features will need invalid scenarios that can be added to the same feature file, this is how it might look like:</p>

```ruby
# spec/features/user_submits_a_link_spec.rb
context "the form is invalid" do
  scenario "they see a useful error message" do
  link_title = "This Testing Rails book is awesome!"
  visit root_path
  click_on "Submit a new link"
  fill_in "link_title", with: link_title
  click_on "Submit!"
  expect(page).to have_content "Url can't be blank"
  end
end
```
<h2>Feature 2: A link appears in the homepage</h2>
<p>Following we will test if the links we have submitted in the previous feature is displayed in the homepage of our app</p>
<p>Again we'll start with some pseudocode</p>

```ruby
As a user
Given a link has already been submitted
When I visit the home page
Then I should see the links title on the page
And it should link to the correct URL
```
<p>For practicality we are going to assume that the link has been submitted in the previous step, so we'll create one in the seeds.rb: </p>

```ruby
link = Link.create(title: "Testing Rails at El Taco Lab", url: "http://eltacolab.com")
```

<h3>Fixtures</h3>

<p> We will use fixtures for creating sample data that we can reuse through the tests</p>

```ruby
# fixtures/links.yml
testing_rails:
title: Testing Rails
url: http://testingrailsbook.com
# In your test
link = links(:testing_rails)
```

<p>As applications grow, you’ll typically need variations on each of your models for
different situations. For example, you may have a fixture for every user role in
your application, then even more users for different roles depending on whether
or not the user is a member of a specific organization</p>

<p>This is how our test code would look like for viewing the links in the home page (For creating the test data we used factorygirl gem)</p>

```ruby
# spec/features/user_upvotes_a_link_spec.rb
RSpec.feature "User upvotes a link" do
  scenario "they see an increased score" do
    link = create(:link)
    visit root_path
    within "#link_#{link.id}" do
    click_on "Upvote"
    end
    expect(page).to have_css "#link_#{link.id} [data-role=score]", text: "1"
  end
end
```

<h2>Instance Models Tests</h2>
<h3>Upvote method</h3>
<p>For this model we will thest the methode #upvote which will allow users to upvote the links posted</p>
<p>Of course it is advised to write pseudo code before writing the test.</p>

```ruby
# spec/models/link_spec.rb
RSpec.describe Link, "#upvote" do
  it "increments upvotes" do
    link = build(:link, upvotes: 1)
    link.upvote
    expect(link.upvotes).to eq 2
  end
end
```

<h3>Score method</h3>
<p>Now let's write the test for a score method where it should return the number of upvotes and downvotes summed to the total score</p>

```ruby
# spec/models/link_spec.rb
RSpec.describe Link, "#score" do
  it "returns the upvotes minus the downvotes" do
    link = Link.new(upvotes: 2, downvotes: 1)
    expect(link.score).to eq 1
  end
end
```
<h2>Validations</h2>

<p>An important test is to check the presence of some attibutes opf models. For that we will use a gem called: shoulda-matchers : </p>

```ruby
gem "shoulda-matchers"
```
<p>After Bundeling we can use the built in matchers from the gem in our tests: </p>

```ruby
RSpec.describe Link, "validations" do
  it { is_expected.to validate_presence_of(:title) }
  it { is_expected.to validate_presence_of(:url) }
  it { is_expected.to validate_uniqueness_of(:url) }
end
```
<p>Of course these validations can also be used for associations or dependencies.</p>

<h2>Mailer tests</h2>

```ruby
# spec/controllers/links_controller_spec.rb
context "when the link is valid" do
  it "sends an email to the moderators" do
    valid_link = double(save: true)
    allow(Link).to receive(:new).and_return(valid_link)
    allow(LinkMailer).to receive(:new_link)
    post :create, link: { attribute: "value" }
    expect(LinkMailer).to have_received(:new_link).with(valid_link)
  end
end
```

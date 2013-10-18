Growable UITextView in a UITableView on iOS7
============================================

There is a lot of activity on the Web about how to include a UITextView inside a UITableView, so that the UITextView does grow and shrink as text is typed in or deleted. However, it is difficult to find a definitive answer, especially for iOS7 which is new. Here is a little guide that tries to walk you through the mysteries of creating such a UITextView. It is, after all, quite simple in the end, but it did take quite a lot of googling to get right.

First, let's create a new UITableViewCell that will hold our UITextView. This is achieved easily in RubyMotion. You might experience some perplexity if you are coming from Objective-C only, but you should be able to quickly understand the gist of the idea.

	class MyUITableViewCell < UITableViewCell
	  def textView
	    @textView
	  end
	  
	  def value
	    @textView.text
	  end
	  
	  def value=(newValue)
	    @textView.text = newValue
	  end
	  
	  def clear
	    @textView.text = nil
	  end
	  
	  def initWithStyle(style, reuseIdentifier:reuseIdentifier)
	    if super then
	      @textView = UITextView.alloc.initWithFrame(self.bounds).tap do |t|
	        t.font = UIFont.systemFontOfSize(UITableViewCell.alloc.init.textLabel.font.pointSize)
	        t.textColor = UIColor.blackColor
	        t.backgroundColor = UIColor.clearColor
	        t.autocorrectionType = UITextAutocorrectionTypeNo
	        t.autocapitalizationType = UITextAutocapitalizationTypeNone
	        t.textAlignment = UITextAlignmentLeft
	        t.enabled = true
	        t
	      end
	      addSubview(@textView)
	  
	      selectionStyle = UITableViewCellSelectionStyleNone
	      accessoryType = UITableViewCellAccessoryNone
	    end
	    
	    self
	  end

Next, let's create our UITableView:

	class MyUITableViewController < UITableViewController
	  def viewDidLoad
	    view.dataSource = view.delegate = self
	  end

	  def viewWillAppear(animated)
	  	@textCell ||= MyIUTableViewCell.alloc.initWithStyle(UITableViewCellStyleDefault, reuseIdentifier:CellID).tap do |c|
	  	  c.textView.delegate = delegate #to receive the textViewDidChange events (option 1)
	      c.value = valueForKey(property)
	      c
	    end
	    setEditing(true, animated:true)
	  end

	  def tableView(tableView, numberOfRowsInSection:section)
	    1
	  end

	  def tableView(tableView, cellForRowAtIndexPath:indexPath)
	    @textCell
	  end



Option 1 - textViewDidChange
============================

We have two options to make the UITextView grow as text is typed in. I started with the first, but eventually moved to the second one, which is more elegant. I keep the first option here for reference, and because it does include some interesting code.

First, let's create our UITableView. We need this code whatever the option we select

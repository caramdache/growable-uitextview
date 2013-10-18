Growable UITextView in a UITableView on iOS7
============================================

There is a lot of activity on the Web about how to include a UITextView inside a UITableView, so that the UITextView does grow and shrink as text is typed in or deleted. However, it is difficult to find a definitive answer, especially for iOS7 which is new. Here is a little guide that tries to walk you through the mysteries of creating such a UITextView. It is, after all, quite simple in the end, but it did take quite a lot of googling to get right.

First, let's create a new UITableViewCell that will hold our UITextView. This is achieved easily in RubyMotion. You might experience some difficulties if you are versed on Objective-C only, but you should be able to quickly get the gist of the idea.

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

Next, let's create our UITableView. We want the delegates to bet set properly and the cell should be in editing mode so the UITextView can be typed in.

	class MyUITableViewController < UITableViewController
	  def viewDidLoad
	    view.dataSource = view.delegate = self
	  end

	  def viewWillAppear(animated)
	  	@textCell ||= MyIUTableViewCell.alloc.initWithStyle(UITableViewCellStyleDefault, reuseIdentifier:CellID).tap do |c|
	  	  c.textView.delegate = delegate #to receive the textViewDidChange events (option 1)
	      c.value = "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi. Nam liber tempor cum soluta nobis eleifend option congue nihil imperdiet doming id quod mazim placerat facer possim assum. Typi non habent claritatem insitam; est usus legentis in iis qui facit eorum claritatem. Investigationes demonstraverunt lectores legere me lius quod ii legunt saepius. Claritas est etiam processus dynamicus, qui sequitur mutationem consuetudium lectorum. Mirum est notare quam littera gothica, quam nunc putamus parum claram, anteposuerit litterarum formas humanitatis per seacula quarta decima et quinta decima. Eodem modo typi, qui nunc nobis videntur parum clari, fiant sollemnes in futurum."
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

We also want the keyboard to slide out when return is pressed. This is achieved as usual.

	class MyUITableViewController
	  def textFieldShouldReturn(textField)
	    textField.resignFirstResponder
	    return true
	  end

Next, we want the need to tell the UITableView that our cell has an unordinary height.

	class MyUITableViewController
	  def tableView(tableView, heightForRowAtIndexPath:indexPath)
	    @textCell.height
	  end

How to compute the height? You would be surprised how many answers have been posted on the Web to such a simple problem. Let's use the following solution, which is simple, elegant, correct and modern (some people use contentSize, but in my experience this does not work reliably). We will put this method in MyUITableViewCell.

	class MyUITableViewCell
	  def height
	    textViewWidth = textView.frame.size.width
	    size = textView.sizeThatFits(CGSizeMake(textViewWidth, Float::MAX))
	    size.height
	  end  

We are now ready to see how to grow the cell as text is typed in.

Option 1 - textViewDidChange
============================

We have two options to make the UITextView grow as text is typed in. I started with the first, but eventually moved to the second one, which is more elegant. I keep the first option here for reference, and because it does include some interesting code.

The method to hookup is textViewDidChange, which takes the textView to grow as an argument. Growing is performed simply by setting the UITextView's frame height to the height of its content. Simple isn't it?

	class MyUITableViewController
	  def textViewDidChange(textView)
	    UITextView.beginAnimations(nil, context:nil)
	    UITextView.setAnimationDuration(0.5)
	    self.tableView.beginUpdates

	    frame = textView.frame
	    frame.size.height = textViewHeight(textView) + 10.0 # add some padding
	    textView.frame = CGRectZero # to avoid text clipping
	    textView.frame = frame

	    self.tableView.endUpdates
	    UITextView.commitAnimations
	  end

There are some extra complications however as you can see. First, you need to tell the UITableView that its content has changed so it has an opportunity to redraw. This is achieved by simply creating a UITableView transaction (begin/endUpdates).

Second, we need to buffer the animation to avoid the screen flickering. This is done by surrounding the transaction by a begin/commitAnimations.

Third, we reset the frame before we update it, to avoid the text to be clipped on screen. Do not ask me why, this is probably a bug in iOS7 and it took me considerable time to get rid of.


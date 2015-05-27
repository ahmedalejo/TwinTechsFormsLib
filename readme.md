# TwinTechs Xamarin Forms Library

This project contains useful helpers, services and controls which aid development of Xamarin Forms apps.

We make these freely available to the community under the apache License [http://www.apache.org/licenses/LICENSE-2.0.html]. Use it how you like.

We are in the process of adding more controls to this library, and will shortly create an official nuget package.

##Xamarin Forms
We will contribute some of these controls to the xamarin forms project once we do more testing/development and have more time to integrate them. We figured it'd be great to get these out there for others to use, and build their ideas upon.

##Caveat emptor (buyer beware)
This library is in an alpha stage. Do not use in a production environment unless you know what you are doing, know what these controls are doing, and have budget to investigate and contribute to resolving any issues which may arise.

#Controls

## FastCell

### Overview
This is a ViewCell replacement base class which has some optimizaitons which drastically improve performance of listview cells, when using xaml. 

### The problem
Currently xamarin forms creates a view cell for each item in a list, even though the underlying template will only use as many cells as are visible on the screen). If your view cell is based on xaml, this means your constructor will call InitializeComponent as many times as you have items. This is what causes the jumping and stuttering when you use Xaml.



### The solution

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/33ZeU1X2M2Y/0.jpg)](https://www.youtube.com/watch?v=33ZeU1X2M2Y)

The optimizaitons work by only initilaizing the xamarin cells for the actual cells which are used. The code changes required to achieve this are minimial.

iOS example [http://youtu.be/V3Djg3bZN1A](http://youtu.be/V3Djg3bZN1A)

And exanded android example here :[http://youtu.be/6yzXOGG2Rm0](http://youtu.be/6yzXOGG2Rm0)


### How to use

#### Extend FastCell, and initialize your cellat the appropriate place
Remove your expensive contstructor from your xaml page and replace with an implementation of the abstract method InitializeCell:

		protected override void InitializeCell ()
		{
			InitializeComponent ();
		}
		
Note, you can also do other initialization here if you need.

#### Set data your cell up with the data in the BindingContext
We don't use bindings in our cells, we instead opt to set the data values by hand, when the binding context changes.

Just override the abstract SetupCell method, and you're good to go:

	protected override void SetupCell (bool isRecycled)
		{
			var mediaItem = BindingContext as MediaItem;
			if (mediaItem != null) {
				UserThumbnailView.ImageUrl = mediaItem.ImagePath ?? "";
				ImageView.ImageUrl = mediaItem.ThumbnailImagePath ?? "";
				NameLabel.Text = mediaItem.Name;
				DescriptionLabel.Text = mediaItem.Description;
			}
		}
		
		
#### Clean up the cache when you finish with your list

When you want to destory your list, destroy the list cache objects by calling : 

    FastCellCache.FlushAllCaches ();
    
Manage references to the FastCellCache instance (which is a singleton) by whatever means you like (injection, static helper method, etc).

i.e. In the example core project, you will find the following class:

		public static class AppHelper
		{
			public static IFastCellCache FastCellCache { get; set; }
		}
	
The reference is set in the appdelegate/main activity, and used in on dissapear.

We will improve the cache situation over the next few weeks, as we do more work on android projects.


### Implementation notes
The following work:

  * Xaml cells,
  * xaml bindings,
  * Grouped cells,
  * Android and iOS implementations (for android you need xamarin forms 1.4.3 (pre2 at time of writing), due to a xamarin forms bug (you'll get crashes otherwise)).
  
Variable height rows do not work, and will not work unless Xamarin provide us some API hooks. If you want to see this performance on xaml cells, with variable heights, please make your voice heard here : [https://bugzilla.xamarin.com/show_bug.cgi?id=29820](https://bugzilla.xamarin.com/show_bug.cgi?id=29820)

The implementations have not been tested for memory leaks yet!


## FastImage
A control which provides ram/disk caching of network loaded images, to give great performance improvements in lists and for image use in genera. 

### Implementation notes
  * On iOS we use SDWebImage, 
  * on Android we use MonoDroidToolkit's ImageLoader on Android [https://github.com/jamesmontemagno/MonoDroidToolkit](https://github.com/jamesmontemagno/MonoDroidToolkit)

Usage 

	UserThumbnailView.ImageUrl = mediaItem.ImagePath ?? "";

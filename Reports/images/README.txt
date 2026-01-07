Images files (.png or .jpg format) placed in this directory are available for 
use within templates. They will be converted to base64 and embedded within the
document rather than loaded as separate image files.

Ensure image filenames do not contain spaces. Given a filename image1.png, you
can include this within a template with the following:

<img src="data:image/png;base64,[% images.image1 %]" />

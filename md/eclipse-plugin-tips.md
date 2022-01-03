
###  使用icon图标
```java
package com.leaderli.visual.editor.util;

import java.net.MalformedURLException;
import java.net.URL;

import org.eclipse.jface.resource.ImageDescriptor;
import org.eclipse.jface.resource.ImageRegistry;
import org.eclipse.swt.graphics.Image;

import com.leaderli.visual.editor.Activator;
import com.leaderli.visual.editor.constant.Leaderli;


public class ImageUtil {

	public static final ImageRegistry IMAGE_REGISTRY = Activator.getDefault().getImageRegistry();

	public static Image getImage(String name) {

		getImageDescriptor(name);
		return IMAGE_REGISTRY.get(name);
	}

	public static ImageDescriptor getImageDescriptor(String name) {

		ImageDescriptor imageDescriptor = IMAGE_REGISTRY.getDescriptor(name);
		if (imageDescriptor == null) {
			try {
				imageDescriptor = ImageDescriptor
						.createFromURL(new URL(Activator.getDefault().getBundle().getEntry("/"), Leaderli.ICON_PATH + name));
				IMAGE_REGISTRY.put(name, imageDescriptor);
			} catch (MalformedURLException e) {
				e.printStackTrace();
				
			}
		}
		return imageDescriptor;
	}
}

```
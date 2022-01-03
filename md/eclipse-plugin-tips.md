
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

## 进度条
参考 [Eclipse Jobs and Background Processing - Tutorial](https://www.vogella.com/tutorials/EclipseJobs/article.html)

```java
import java.util.concurrent.TimeUnit;

import org.eclipse.core.runtime.IProgressMonitor;
import org.eclipse.core.runtime.IStatus;
import org.eclipse.core.runtime.Status;
import org.eclipse.core.runtime.SubMonitor;
import org.eclipse.core.runtime.jobs.Job


Job job = new Job("create project : " + projectName) {

		@Override
		protected IStatus run(IProgressMonitor monitor) {
			try {
				//将任务分为100个
				SubMonitor subMonitor = SubMonitor.convert(monitor, 100);
				for (int i = 0; i < 10; i++) {
					//完成10个
					subMonitor.split(10);
					monitor.setTaskName("create project:" + i);
					// 设定为还剩70个
					// subMonitor.setWorkRemaining(70);
					TimeUnit.SECONDS.sleep(1);
				}

			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			return new Status(IStatus.OK, Activator.PLUGIN_ID, Messages.status_OK);
		}
};
job.schedule();



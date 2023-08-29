# Project Retrospective

Here are some learnings from the project that I currently have, due to the NDA, I can't share the whole project, but I can share some of the learnings and part of the codes that I have from the project.

<details>

<summary>Next.js</summary>

## Dynamic Metadata

- [Metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)

Metadata is the API that can be used to describe the content of a page. It is used by search engines, social media, and messaging services to get a preview of the content.

In Next.js, there are two ways to add metadata to a page:

- Config-based Metadata.(We were using this one)
- File-based Metadata.

## The Metadata object

- [Metadata Object and generateMetadata Options](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)

Define Metadata Object from `layout.tsx` file.

There are some fields that are required for the Metadata object:

- `title`
- `template`(optional): Can be added a prefix or a suffix to titles defined in child routes.
- `description`(optional)
- `default` - A fallback title to child routes that don't have their own title.
- `absolute`: Provide a title that ignores `title.template`.

## Project retro

When I tried to implement dynamic metadata, there was an issue that I needed to fix:

- my root `layout.tsx` was not server-side due to the `BottomNavbar.tsx` component that needed to be detected in order to show the bottom navbar.

### How to fix it?

Previously, `BottomNavbar.tsx` was an uncontrolled component, since I needed my root `layout.tsx` to be server-side, I've decided to make `BottomNavbar.tsx` a controlled component.

essentially, the `BottomNavbar` would be hidden when the user scrolls down, and it would be shown when the user scrolls up. Here the states were passed from the root `layout.tsx` as props to the `BottomNavbar.tsx` component.

```typescript
// layout.tsx
// BottomNavbar: Uncontrolled component

"use client";

import * as React from "react";

const workSans = Work_Sans({ subsets: ["latin"] });

export default function RootLayout({
	children,
	params: { lang },
}: {
	children: React.ReactNode;
	params: ServerSidePageType["params"];
}) {
	const pathName = usePathname();
	const [scrollingDown, setScrollingDown] = React.useState(true);
	const [navbarVisible, setNavbarVisible] = React.useState(true);

	const timeoutIdRef = React.useRef<NodeJS.Timeout | null>(null);

	React.useEffect(() => {
		const handleScroll = () => {
			const isScrollingDown = window.scrollY > (scrollY || 0);
			setScrollingDown(isScrollingDown);

			if (timeoutIdRef.current) {
				clearTimeout(timeoutIdRef.current);
			}

			timeoutIdRef.current = setTimeout(() => {
				const shouldShowNavbar = isScrollingDown || window.scrollY === 0;
				setNavbarVisible(shouldShowNavbar);
			}, 150);
		};

		window.addEventListener("scroll", handleScroll);

		return () => {
			window.removeEventListener("scroll", handleScroll);
			if (timeoutIdRef.current) {
				clearTimeout(timeoutIdRef.current);
			}
		};
	}, []);

	return (
		<html lang={lang} dir={dir(lang)}>
			<body className={`${workSans.className} relative bg-MainBG`}>
				<ReduxProvider>
					<PopUpBanner />
					<Header lang={lang} />
					{navbarVisible && (
						<BottomNavbar path={pathName} scrollDown={scrollingDown} />
					)}
					{children}
					<Footer lang={lang} hideFooter={false} />
				</ReduxProvider>
			</body>
		</html>
	);
}
```

### Making BottomNavbar a controlled component

```typescript
// layout.tsx
// Change to Server-side

const workSans = Work_Sans({ subsets: ["latin"] });

export default function RootLayout({
	children,
	params: { lang },
}: {
	children: React.ReactNode;
	params: ServerSidePageType["params"];
}) {
	return (
		<html lang={lang} dir={dir(lang)}>
			<body className={`${workSans.className} relative bg-MainBG`}>
				<ReduxProvider>
					<PopUpBanner />
					<Header lang={lang} />
					<BottomNavbar />
					{children}
					<Footer lang={lang} hideFooter={false} />
				</ReduxProvider>
			</body>
		</html>
	);
}
```

```typescript
// BottomNavbar.tsx
"use client";

import * as React from "react";

export const BottomNavbar = () => {
  const path = usePathname();
  const [scrollingDown, setScrollingDown] = React.useState(true);
  const [navbarVisible, setNavbarVisible] = React.useState(true);

  const timeoutIdRef = React.useRef<NodeJS.Timeout | null>(null);

  React.useEffect(() => {
    const handleScroll = () => {
      const isScrollingDown = window.scrollY > (scrollY || 0);
      setScrollingDown(isScrollingDown);

      if (timeoutIdRef.current) {
        clearTimeout(timeoutIdRef.current);
      }

      timeoutIdRef.current = setTimeout(() => {
        const shouldShowNavbar = isScrollingDown || window.scrollY === 0;
        setNavbarVisible(shouldShowNavbar);
      }, 150);
    };

    window.addEventListener("scroll", handleScroll);

    return () => {
      window.removeEventListener("scroll", handleScroll);
      if (timeoutIdRef.current) {
        clearTimeout(timeoutIdRef.current);
      }
    };
  }, []);

  const maximumNum = (amount: number) => {
    ...
  };
  const location = path.split("/")[2];



  return (
    <>
      {navbarVisible ? (
        <nav
          className={`${
            location === "maintenance" || (location === "error-oops" && scrollingDown) ? "hidden" : "visible fixed"
          } bottom-0 z-[999] w-full bg-primaryGold md:hidden `}
        >
          <ul className="flex justify-between px-[18px] pt-[16px] sm:justify-around">
            ...links
          </ul>
        </nav>
      ) : null}
    </>
  );
};
```

Now my `layout.tsx` was server-side, and I was able to implement dynamic metadata.

Example of using dynamic metadata:

```typescript
// layout.tsx

export const metadata: Metadata = {
	title: {
		default: "Brand",
		template: "%s | Brand Name",
	},
	description: "...",
	keywords: "...",
};
```

```typescript
// terms-condition/page.tsx

export const metadata: Metadata = {
	title: "Terms & Conditions",
};
```

The result would be `Terms & Conditions | Brand Name`.

</details>

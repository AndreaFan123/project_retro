# Project Retrospective

Here are some learnings from the project that I currently have, due to the NDA, I can't share the whole project, but I can share some of the learnings and part of the codes that I have from the project.

<details>

<summary>Next.js: Dynamic Metadata</summary>

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

- my root `layout.tsx` was not server component due to the `BottomNavbar.tsx` component that needed to be detected in order to show the bottom navbar.

### How to fix it?

Previously, `BottomNavbar.tsx` was an uncontrolled component, since I needed my root `layout.tsx` to be server component, I've decided to make `BottomNavbar.tsx` a controlled component.

essentially, the `BottomNavbar` would be hidden when the user scrolls down, and it would be shown when the user scrolls up. Here the states were passed from the root `layout.tsx` as props to the `BottomNavbar.tsx` component.

```typescript
// layout.tsx: Client component
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
// Change to Server component by not specifying "use client"

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

Now my `layout.tsx` was server component, and I was able to implement dynamic metadata.

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

<details>
<summary>Next.js: Fetch API</summary>

</details>

<details>
  <summary> Complicated Form </summary>

## Design a input component that can be applied to different situations

### The problem

When it comes to form, there are many different types of components. In order to design dynamic components that can be applied to multiple scenario, we need to consider the following things:

- Types of input(checkbox, radio, text, number, email, password, etc.)
- Warning / Success messages / icons
- States of input / select / buttons, etc (focus, valid, invalid, disabled, etc.)
- Actions of input (onchange, onblur, onfocus, etc.)

Following were the input / select / buttons, etc, `registration information` that required user to fill in in my project:

- title (chip when big screen, select when small screen)
- First name (input)
- Last name (input)
- Email (input)
- confirm email (input)
- mobile with no country codes. (input)
- Country (selector)
- Preferred language (chip)
- gender (chip)
- Birthday Month (selector)
- Birthday Year (selector)

But there won't be only one form in the project, there will be many different forms, following were the other forms that required user to fill in:

`Contact form`:

- title (chip when big screen, select when small screen)
- First name (input)
- Last name (input)
- Email (input)
- Mobile with default country code (input)
- member number (input)
- Enquiry (selector)
- Details of enquiry (input)
- Message (textarea)
- image upload (input)

`Campaign form`:

- title (chip when big screen, select when small screen)
- First name (input)
- Last name (input)
- Mobile with default country code (input)
- Email (input)
- Type of event (selector)
- Event Date (input)
- Event Time (hr and min) (selector)
- No. of Guests (chip)
- Company Name (input)
- Budget (input)
- Message (textarea)

## Input

A standard input would be like this:

```html
<label for="">XXX</label> <input id="" placeholder="" type="" />
```

Or in React:

```javascript
<label htmlFor="">XXX</label>
<input id="" placeholder="" type="" />
```

## Prepare constants

There are two ways to prepare constants:

- Make an `array` of inputs with series of information in it.
- Make multiple `objects` of inputs with series of information in it.

First of all, let's define types/interface.

### Here was my original types/interface:

```typescript
export interface RegistrationProps {
	labelText?: string;
	labelFor: string;
	id?: string;
	name: string;
	type: string;
	autoComplete: string;
	isRequired: boolean;
	placeHolder?: string;
	isEmailMatching?: boolean;
	isValidEmail?: boolean;
	isFirstNameValid?: boolean;
	isLastNameValid?: boolean;
	isMobileValid?: boolean;
	isMemberDigitValid?: boolean;
	maxLength?: number;
	path?: string;
	lang?: string;
	storeValue?: string;
	onChange?: (e: React.ChangeEvent<HTMLInputElement>) => void;
}
```

### This is the revised types/interface:

```typescript
// types.ts

import

type InputType =
	| "text"
	| "email"
	| "password"
	| "number"
	| "tel"
	| "checkbox"
	| "radio";

type InputAutoComplete = "on" | "off";

// Input type

export interface Input {
	labelFor: string;
	labelText: string;
	placeholder: string;
	type: InputType;
	autoComplete: InputAutoComplete;
	required: boolean;
	name: string;
	id: string;
	path?: string;
	errorMsg?: string;
	successMsg?: string;
	disabled?: boolean;
	minLength: number;
	maxLength: number;
	onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
	onKeyDown: (e: React.KeyboardEvent<HTMLInputElement>) => void;
	storeValue?: string;
	innerRef: React.RefObject<HTMLInputElement>;
	isValid?: boolean;
	value: string;
}
```

### What have I changed?

- I've changed the `type` and `autoComplete` to `enum` type.
- I removed `isEmailMatching`, `isFirstNameValid`, `isLastNameValid`, `isMobileValid`, `isMemberDigitValid` as I can just create states to store the values of each input and assign them to `isValid` property.
- Added `onKeyDown` and `innerRef` property to the interface.

With the types/interface defined, we can prepare constants now.

### Here was my original input constants:

```typescript
export const firstName: RegistrationProps = {
	labelText: "Your Name*",
	labelFor: "first-name",
	id: "first-name",
	name: "firstName",
	type: "text",
	autoComplete: "name",
	isRequired: true,
	placeHolder: "First Name",
	path: "",
	lang: "",
	onChange: () => undefined,
};
```

### This is the revised input constants:

```typescript
// form.ts

export const firstName: Input = {
	labelFor: "firstName",
	labelText: "Your Name*",
	placeholder: "First Name",
	type: "text",
	autoComplete: "on",
	required: true,
	name: "firstName",
	id: "firstName",
	errorMsg: "Incorrect format",
	successMsg: "",
	disabled: false,
	minLength: 0,
	maxLength: 25,
	onChange: (e: React.ChangeEvent<HTMLInputElement>) => {
		e;
	},
	onKeyDown: (e: React.KeyboardEvent<HTMLInputElement>) => {
		e;
	},
	storeValue: "",
	innerRef: React.createRef<HTMLInputElement>(),
	isValid: false,
	value: "",
};

// Repeat the same for other inputs
```

### Implement constants in the form

Inside our form, we can use the constants like this: I used to create states for each input, but it was not a good practice, using object to store the values of each input is a better way.

```typescript
// RegistrationForm.tsx
import * as React from "react";
import { firstName, lastName } from "...";

export const RegistrationForm = () => {
	const [inputVal, setInputVal] = React.useState({
		firstName: "",
		lastName: "",
		email: "",
		mobile: "",
	});
	return (
		<form>
			<InfoInput2
				labelFor={firstName.labelFor}
				labelText={firstName.labelText}
				placeholder={firstName.placeholder}
				type={firstName.type}
				autoComplete={firstName.autoComplete}
				required={firstName.required}
				name={firstName.id}
				id={firstName.id}
				errorMsg={firstName.errorMsg}
				minLength={firstName.minLength}
				maxLength={firstName.maxLength}
				isValid={isFistNameValid}
				onChange={handleInputChange}
				onKeyDown={(e: React.KeyboardEvent<HTMLInputElement>) =>
					handleKeyDown(e, 0)
				}
				value={inputVal.firstName}
				innerRef={inputRefs.current[0]}
			/>
		</form>
	);
};
```

## My spaghetti input component

This was my initial `input` component, it has some basic functionalities, which are
:

- It will show warning icon and error message when user enters invalid input.
- It will show check icon when user enters valid input.

### Breakdown the component

```typescript
import * as React from "react";

// Create a component that takes in props and render icon accordingly.
const showIcons = (condition: boolean, src: StaticImageData, alt: string) => {
	if (condition) {
		return (
			<Image
				src={src}
				width={20}
				height={0}
				alt={alt}
				className="absolute right-[25px] top-[18px] self-center md:right-[30px] 2xl:right-[20px]"
			/>
		);
	}
};

// Create a component that takes in props and render warning message accordingly.
const showWarning = (condition: boolean, str: string) => {
	if (condition) {
		return (
			<span className="block pl-4 pt-1 text-[14px] font-semibold leading-4 text-primary">
				{str}
			</span>
		);
	}
};

export default function InfoInput({
	labelText,
	labelFor,
	id,
	name,
	type,
	isRequired,
	placeHolder,
	autoComplete,
	onChange,
	isEmailMatching,
	isValidEmail,
	isFirstNameValid,
	isLastNameValid,
	isMobileValid,
	isMemberDigitValid,
	maxLength,
	path,
	lang,
	storeValue,
}: RegistrationProps) => {
  const [value, setValue] = React.useState("");

  const handleValue = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
    if (onChange !== undefined) onChange(e);
  };

  return (
    <>
     {labelText !== undefined && (
        <label
          htmlFor={labelFor}
          className="text-[14px] font-semibold leading-7 text-primaryDark xl:text-[16px] xl:text-primaryGold"
        >
          {labelText}
        </label>
         <input
          maxLength={maxLength}
          className="relative mb-[-10px] block w-full rounded-full border-[1px]  px-[20px] py-[12px] text-[16px] font-medium placeholder-primaryGold placeholder-opacity-40 focus:border-primaryGold focus:outline-none focus:ring-1 focus:ring-primaryGold md:h-[48px] md:py-[20px] md:text-[18px] xl:py-[24px]"
          id={id}
          name={name}
          type={type}
          required={isRequired}
          placeholder={placeHolder}
          autoComplete={autoComplete}
          value={storeValue || value}
          onChange={handleValue}
        />
         {value.length > 0 && (
          <>
            {id === "email" && showIcons(isValidEmail === false, Warning, "Wrong email")}
            {id === "email" &&
              showIcons((isValidEmail !== undefined && isValidEmail), Check, "Correct email")}
            {id === "confirm-email" && showIcons(isEmailMatching === false, Warning, "Wrong email")}
            {id === "confirm-email" &&
              showIcons(
                (isEmailMatching !== undefined && isEmailMatching),Check,"Correct email")}
            {id === "first-name" &&
              showIcons(
                (isFirstNameValid !== undefined && isFirstNameValid),Check,"Correct format of first name")}
            {id === "first-name" && showIcons(isFirstNameValid === false, Warning, "Wrong format of last name")}
            {id === "last-name" && showIcons((isLastNameValid !== undefined && isLastNameValid),Check,"Correct format of last name")}
            {id === "last-name" && showIcons(isLastNameValid === false, Warning, "wrong format of last name")}
            {id === "mobile" && showIcons((isMobileValid !== undefined && isMobileValid),Check,"Correct format of mobile")}
            {id === "mobile" && showIcons(isMobileValid === false, Warning, "wrong format of mobile")}
          </>
        )}
        </>
  )
}
```

### Revised the code

```typescript
export const InfoInput2 = ({
	labelFor,
	labelText,
	placeholder,
	type,
	autoComplete,
	required,
	name,
	id,
	errorMsg,
	minLength,
	maxLength,
	onChange,
	onKeyDown,
	value,
	isValid,
	innerRef,
}: Input) => {
	const fieldIds = ["firstName", "lastName", "email", "confirmEmail", "mobile"];

	const renderIconsConditionally = (fieldId: string, index: number) => {
		if (id === fieldId && value.length > 0) {
			return (
				<RenderIcons
					key={index}
					isValid={isValid !== undefined && isValid}
					errorMsg={id === fieldId && isValid === false ? errorMsg : undefined}
				/>
			);
		}
	};

	return (
		<div className="relative flex w-full flex-col gap-1">
			<label
				htmlFor={labelFor}
				className="block text-[14px] font-semibold leading-5 lg:text-primaryGold"
			>
				{labelText}
			</label>
			<input
				id={id}
				name={name}
				placeholder={placeholder}
				type={type}
				autoComplete={autoComplete}
				required={required}
				minLength={minLength}
				maxLength={maxLength}
				onChange={onChange}
				onKeyDown={onKeyDown}
				ref={innerRef}
				className={`rounded-full border bg-transparent px-[12px] pl-[20px] pt-[12px] placeholder-primaryGold placeholder-opacity-40 placeholder:pl-[8px] focus:bg-transparent focus:outline-none focus:ring-1 lg:placeholder:text-[16px] ${
					value.length > 0 && isValid !== undefined && isValid === false
						? "border-primary focus:border-primary focus:ring-primary"
						: "border-primaryGold focus:border-primaryGold focus:ring-primaryGold"
				}`}
			/>
			{fieldIds.map((fieldId, index) =>
				renderIconsConditionally(fieldId, index)
			)}
		</div>
	);
};
```

### What have I changed?

- I've removed the `showIcons` and `showWarning` functions, and created a new component called `RenderIcons` to render icons and warning messages.
- I've removed the `handleValue` function, and used `onChange` to handle the value of each input.
- Create an object called fieldIds, which stores the ids of each input.
- Create `renderIconsConditionally` function to render icons and warning messages conditionally.

---

## Chip

Chip is a component that can be used to select multiple options. It is usually used in the form to select multiple options, but in my project, it was used to select one option.

We can use button as a chip, let's define the types/interface first.

### Here was my original types/interface:

```typescript
export interface ChipProps {
	title: string;
	items: ChipItem[];
}

export interface ChipItem {
	id: string;
	name?: string;
	label?: string;
	icon?: ReactNode;
	tabIndex: number;
}
```

### This is the revised types/interface:

```typescript
import { ReactNode } from "react";

export interface ChipItem {
	id: string;
	name?: string;
	label?: string;
	icon?: ReactNode;
	tabIndex: number;
}

export interface ChipProps {
	title: string;
	path?: string;
	lang?: string;
	items: ChipItem[];
	onClick: (e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => void;
}
```

### What have I changed?

- I've added `path`, `lang` and `onClick` property to the interface.

### Prepare constants

```typescript
import { ChipProps } from "@/types/registration/chip";

export const LanChips = {
	title: "Preferred Language*",
	onClick: (e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => {
		e.preventDefault();
	},
	items: [
		{
			id: "zh_TC",
			name: "preferredLan",
			label: "繁中",
			tabIndex: 1,
		},
		{
			id: "en",
			name: "preferredLan",
			label: "English",
			tabIndex: 2,
		},
	],
} as ChipProps;

// Repeat the same for other chips
```

### Implement constants in the form

```typescript
// RegistrationForm.tsx
import * as React from "react";
import { firstName } from "...";
import { LanChips } from "...";

export const RegistrationForm = () => {
	const [inputVal, setInputVal] = React.useState({
		firstName: "",
		lastName: "",
		email: "",
		mobile: "",
	});
	return (
		<form>
			<InfoInput
				id={firstName.id}
				labelText={firstName.labelText}
				labelFor={firstName.labelFor}
				name={firstName.name}
				type={firstName.type}
				autoComplete={firstName.autoComplete}
				isRequired={firstName.isRequired}
				placeHolder={firstName.placeHolder}
				isFirstNameValid={isFirstNameValid}
				onChange={handleInput}
				maxLength={25}
				lang={lang}
				path=""
				storeValue=""
			/>
			<Chip
				title={LanChips.title}
				items={LanChips.items}
				onClick={handleButton}
				path={path}
				lang={lang}
			/>
		</form>
	);
};
```

### My spaghetti Chip component

```typescript
// OldChip.tsx

interface Props {
	items: ChipProps;
	storeValue?: string;
	path?: string;
	lang?: string;
	buttonClass?: string;
	onChange: (e: MouseEvent<HTMLButtonElement>) => void;
}

export const Chips = ({
	items,
	onChange,
	buttonClass,
	storeValue,
	path,
	lang,
}: Props) => {
	return (
		<div className="flex w-full flex-col gap-[12px]">
			<label
				htmlFor={items.labelTitle}
				className="text-[14px] font-semibold leading-4 text-primaryDark lg:text-primaryGold xl:text-[16px]"
			>
				{title}
			</label>
			<div className="flex  gap-[12px] md:w-full md:gap-[16px] ">
				{items.items.map((item, index) => (
					<button
						className={`flex items-center justify-center rounded-[16px] border-[1px] border-primaryGold p-[16px] text-[14px] leading-5
            text-primaryDark sm:text-[14px] lg:rounded-[40px]  xl:px-4 xl:py-[.9rem] xl:text-[16px] ${
							selectedIndex === item.tabIndex &&
							"bg-primaryGold font-semibold text-white"
						} 
            ${
							item.label === "male" || item.label === "female"
								? "flex h-[55px] w-[55px] items-center justify-center rounded-full"
								: ""
						}
            ${buttonClass}
            `}
						value={item.label}
						key={index}
						type="button"
						role="button"
						aria-label={item.label}
						tabIndex={item.tabIndex}
						name={item.name}
						onClick={(e) => handleChipSelect(item.tabIndex, e)}
					>
						{item.label !== "male" && item.label !== "female" && (
							<span>{item.label}</span>
						)}
						{item.label === "male" && (
							<Image
								src={
									selectedIndex === item.tabIndex || storeValue === item.label
										? SelectedMale
										: MobileMaleIcon
								}
								width={0}
								height={0}
								alt="Male Icon"
								className="block h-auto w-[10px] self-center"
							/>
						)}
						{item.label === "female" && (
							<Image
								src={
									selectedIndex === item.tabIndex || storeValue === item.label
										? SelectedFemale
										: MobileFemaleIcon
								}
								width={0}
								height={0}
								alt="Female Icon"
								className="block h-auto w-[14px] self-center"
							/>
						)}
					</button>
				))}
			</div>
		</div>
	);
};
```

### Revised the code

```typescript
// NewChip.tsx

const renderMaleOrFemaleIcon = (
	label: string,
	selectIndx: number | null,
	indx: number
) => {
	return (
		<>
			{label === "male" ? (
				<Image
					src={selectIndx === indx ? SelectedMale : MobileMaleIcon}
					width={0}
					height={0}
					alt="Male Icon"
					className="block h-auto w-[10px] self-center"
				/>
			) : (
				<Image
					src={selectIndx === indx ? SelectedFemale : MobileFemaleIcon}
					width={0}
					height={0}
					alt="Female Icon"
					className="block h-auto w-[14px] self-center"
				/>
			)}
		</>
	);
};

export const Chip = ({ title, items, path, lang, onClick }: ChipProps) => {
	const [, setSelectedChip] = useState("");
	const [selectedIndx, setSelectedIndx] = useState<number | null>(null);

	const IconStyle =
		"flex h-[55px] w-[55px] items-center justify-center rounded-full";

	return (
		<>
			<label htmlFor={title}>{title}</label>
			<div className="flex gap-4">
				{items.map((item) => (
					<button
						className={`rounded-[16px] border border-primaryGold p-4 text-[14px] leading-5 text-primaryDark sm:text-[14px] lg:rounded-[40px]  xl:px-4 xl:py-[.9rem] xl:text-[16px] ${
							selectedIndx === item.tabIndex &&
							"bg-primaryGold font-semibold text-white"
						} ${
							item.label === "male" || item.label === "female" ? IconStyle : ""
						} ${item.label === "Undisclosed" && "rounded-[40px]"} `}
						name={item.name}
						id={item.id}
						value={item.label}
						key={item.id}
						aria-label={item.label}
						onClick={(e) => handleChipSelectClick(item.tabIndex, e)}
					>
						{item.label !== "male" && item.label !== "female" && (
							<span>{item.label}</span>
						)}
						{item.label === "male" &&
							renderMaleOrFemaleIcon(item.label, selectedIndx, item.tabIndex)}
						{item.label === "female" &&
							renderMaleOrFemaleIcon(item.label, selectedIndx, item.tabIndex)}
					</button>
				))}
			</div>
		</>
	);
};
```

### What have I changed?

- I've extracted `male` and `female` icons into a function called `renderMaleOrFemaleIcon` and passed the parameters to it.
- Extracted icon style into a variable called `IconStyle`.

</details>

<details>
  <summary>Get Districts</summary>

 ### Context
 When user select region, and call api to get all data, then filter the district accrodingly.
 I've passed regionName as parameter, and use `switch` method to check if name was matched.

```typescript
const getDistrictList = (regionName: string) => {
  switch (regionName) {
    case "Hong Kong Island":
      const hkislandDistricts = regionList && regionList[0].districts.map(district => district.name);
         if (hkislandDistricts !== undefined) {
           return hkislandDistricts;
         }
         break;
    case "Kowloon":
         const kowloonDistricts = regionList && regionList[1].districts.map(district => district.name);
         if (kowloonDistricts !== undefined) {
           return kowloonDistricts;
         }
    case "New Territories":
         const newTerritoryDistricts = regionList && regionList[2].districts.map(district => district.name);
         if (newTerritoryDistricts !== undefined) {
           return newTerritoryDistricts;
         }
    default:
         return [];
     }
```

### The downside of this approach
1. Using none unique identifier name.
2. Case shouldn't be hardcode, as if the api is updated.
Below is revised one.

```typescript
  const getDistrictList = (regionName: string) => {
    if (!regionList) {
      return [];
    }

    const region = regionList.find(region => region.name === regionName);
    const districtList = region && region.districts ? region.districts.map(district => district.name) : [];

    return districtList;
}
```
</details>

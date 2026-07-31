#Termux

termux-setup-storage

pkg install libzip

pkg update && pkg install make clang build-essential

/* SPDX-License-Identifier: BSD-3-Clause */
#ifndef __QDL_H__
#define __QDL_H__

#ifdef _WIN32
#include <malloc.h>
#else
#include <alloca.h>
#endif

#include <stdbool.h>
#include <stdint.h>

#include "patch.h"
#include "program.h"
#include "read.h"
#include <libxml/tree.h>
#include "vip.h"

#define container_of(ptr, typecast, member) ({                  \
	void *_ptr = (void *)(ptr);		                \
	((typeof(typecast) *)(_ptr - offsetof(typecast, member))); })

#define MIN(x, y) ({		\
	__typeof__(x) _x = (x);	\
	__typeof__(y) _y = (y);	\
	_x < _y ? _x : _y;	\
})

#define ROUND_UP(x, a) ({		\
	__typeof__(x) _x = (x);		\
	__typeof__(a) _a = (a);		\
	(_x + _a - 1) & ~(_a - 1);	\
})

#define __unused __attribute__((__unused__))

#define ARRAY_SIZE(x) (sizeof(x) / sizeof((x)[0]))
#define ALIGN_UP(p, size) ({						\
		__typeof__(size) _mask = (size) - 1;			\
		(__typeof__(p))(((uintptr_t)(p) + _mask) & ~_mask);	\
})

#define MAPPING_SZ 128

#define SAHARA_ID_EHOSTDL_IMG	13

enum QDL_DEVICE_TYPE {
	QDL_DEVICE_USB,
	QDL_DEVICE_SIM,
	QDL_DEVICE_QUD,
	/*
	 * Meta-backend: defers transport selection to the wait loop inside
	 * auto_open(), which polls libusb and (on Windows) the QUD SetupAPI
	 * enumeration each tick and binds whichever first reaches an EDL
	 * device. Resolves the UX hazard of an upfront probe timeout where
	 * the user plugs in the cable just after the grace window expires.
	 */
	QDL_DEVICE_AUTO,
};

enum qdl_storage_type {
	QDL_STORAGE_UNKNOWN,
	QDL_STORAGE_EMMC,
	QDL_STORAGE_NAND,
	QDL_STORAGE_UFS,
	QDL_STORAGE_NVME,
	QDL_STORAGE_SPINOR,
};

enum qdl_skipblock_mode {
	QDL_SKIPBLOCK_NONE,
	QDL_SKIPBLOCK_SHA256,
};

enum qdl_opt_extended {
	OPT_SIGNEDDIGESTS = 0x104,
	OPT_NO_SAHARA = 0x105,
};

struct oplus_vip_config {
	char *digest_path;
	char *sign_path;
	char *transfer_xml;
	char *verify_xml;
	char *sha256init_xml;
	char *configure_xml;
	char *partition_xml;
	int skip_transfer;
	int enabled;
};

/*
 * Dynamic OnePlus token auth (oplus_token.c). version mirrors oneplus.py's
 * deviceconfig: 1/2 use the setprojmodel path, 3 uses setprocstart +
 * setswprojmodel. For version 2/3 the model-verify name is @cm, for version 1
 * it is @projid. @program_pk / @program_token are the pk/token pair attached
 * to every program, erase and patch command once authenticated.
 */
enum oplus_auth_mode {
	OPLUS_AUTH_OFF = 0,	/* EDL_OP_AUTH unset/other: never authenticate */
	OPLUS_AUTH_FORCE,	/* EDL_OP_AUTH=1: builtin OnePlus / manual override */
	OPLUS_AUTH_AUTO,	/* EDL_OP_AUTH=auto: decide by device param projid */
};

struct oplus_token_config {
	enum oplus_auth_mode auth_mode;
	int enabled;		/* runtime: confirmed-OnePlus, run the handshake */
	int authenticated;
	int version;
	int do_demacia;
	int serial_explicit;	/* EDL_OP_SERIAL was set by the caller; never override it */
	char projid[8];
	char cm[16];
	char serial[24];
	char cf[8];
	char ato_build[8];
	char flash_mode[8];
	uint64_t device_timestamp;
	char program_pk[17];
	char program_token[1025];
};

struct qdl_device {
	enum QDL_DEVICE_TYPE dev_type;
	int fd;
	size_t max_payload_size;
	size_t sector_size;
	enum qdl_storage_type current_storage_type;
	enum qdl_skipblock_mode skipblock_mode;
	unsigned int slot;
	char serial[64];	/* device serial reported by the transport, empty if none */

	int (*open)(struct qdl_device *qdl, const char *serial);
	int (*read)(struct qdl_device *qdl, void *buf, size_t len, unsigned int timeout);
	int (*write)(struct qdl_device *qdl, const void *buf, size_t nbytes, unsigned int timeout);
	void (*close)(struct qdl_device *qdl);
	/* Optional USB port reset (software replug); NULL on transports that can't. */
	int (*reset)(struct qdl_device *qdl);
	void (*set_out_chunk_size)(struct qdl_device *qdl, long size);
	void (*set_vip_transfer)(struct qdl_device *qdl, const char *signed_table,
				 const char *chained_table);

	struct vip_transfer_data vip_data;

	struct oplus_vip_config oplus_vip;
	struct oplus_token_config oplus_token;
	int auto_reset;
	int oplus_mode;		/* programmer requires a label on read/program/erase */

	/*
	 * Pushback buffer for stream-oriented transports (Windows COM via the
	 * QDLoader driver, virtio-console, ...). When a single read crosses a
	 * Firehose message boundary - typically because the binary payload of
	 * a rawmode response trails the XML envelope in the same read - the
	 * leftover bytes are stashed here and qdl_read() returns them before
	 * pulling more data from the transport.
	 */
	char *pending_buf;
	size_t pending_len;
	size_t pending_off;
};

struct sahara_image {
	char *name;
	void *ptr;
	size_t len;
};

struct qdl_zip;

struct libusb_device_handle;

struct qdl_device *qdl_init(enum QDL_DEVICE_TYPE type);
void qdl_deinit(struct qdl_device *qdl);
int qdl_open(struct qdl_device *qdl, const char *serial);
void qdl_close(struct qdl_device *qdl);
int qdl_read(struct qdl_device *qdl, void *buf, size_t len, unsigned int timeout);
int qdl_push_back(struct qdl_device *qdl, const void *buf, s


make clean && TERMUX=1 make



./qdl /storage/emulated/0/images/DevprgProgrammer2.elf /storage/emulated/0/images/rawprogram*.xml /storage/emulated/0/images/patch*.xml
